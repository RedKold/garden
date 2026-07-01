# OS Lab M8: FAT-32 文件系统恢复

## 1. FAT-32 磁盘布局

一个 FAT-32 分区从 LBA 0 开始，布局如下：

```
+-------------------+-----------------+-------------------+-------------------+--------------------+
|  保留区 (Reserved) |  FAT 表 1 (主)   |  FAT 表 2 (备份)  |  根目录区          |  数据区 (Data)     |
+-------------------+-----------------+-------------------+-------------------+--------------------+
```

### 1.1 引导扇区 (BPB) — LBA 0

第 1 个扇区（512 字节）是引导扇区 + BPB，关键字段：

| 偏移 | 大小 | 字段 | 说明 |
|------|------|------|------|
| 0x00 | 3 | `BS_jmpBoot` | 跳转指令 |
| 0x0B | 2 | `BPB_BytsPerSec` | 每扇区字节数（通常 512） |
| 0x0D | 1 | `BPB_SecPerClus` | 每簇扇区数（通常 1, 2, 4, 8, ...） |
| 0x0E | 2 | `BPB_RsvdSecCnt` | 保留扇区数（通常 32） |
| 0x10 | 1 | `BPB_NumFATs` | FAT 表份数（通常 2） |
| 0x24 | 4 | `BPB_FATSz32` | **每个 FAT 表占用的扇区数** |
| 0x2C | 4 | `BPB_RootClus` | 根目录的起始簇号（通常 2） |
| 0x30 | 2 | `BPB_FSInfo` | FSInfo 扇区号（通常 1） |
| 0x32 | 2 | `BPB_BkBootSec` | 引导扇区备份位置（通常 6） |

### 1.2 计算关键地址

```
FAT1 起始扇区 = BPB_RsvdSecCnt                   # 通常 = 32
FAT2 起始扇区 = FAT1 起始扇区 + BPB_FATSz32
根目录区起始扇区 = FAT2 起始扇区 + BPB_FATSz32
数据区起始扇区 = 根目录区起始扇区（根目录也是一个普通的簇）
```

簇号到扇区的转换：
```
数据区起始 LBA = FAT2_起始 + BPB_FATSz32
簇 N 的起始 LBA = 数据区起始 LBA + (N - 2) * BPB_SecPerClus
```

### 1.3 FAT 表项详解

FAT 表是一个**簇号数组**，每个表项 4 字节（32 位），数组下标就是簇号。

- **FAT[0]**: 介质描述符（低 8 位），其他位为 1
- **FAT[1]**: 文件系统脏标志
- **FAT[2]**: 数据区第一个簇（簇 2，通常也是根目录）

表项值的含义：

| FAT 表项值 | 含义 |
|-----------|------|
| 0x00000000 | 空闲簇 |
| 0x00000002 - 0x0FFFFFEF | 指向下一个簇号（文件的下一个数据块） |
| 0x0FFFFFF0 - 0x0FFFFFF6 | 保留 |
| 0x0FFFFFF7 | 坏簇 |
| 0x0FFFFFF8 - 0x0FFFFFFF | **文件结束 (EOF)** |

**FAT 表的工作方式（链式结构）**：

```
文件 "HELLO.TXT" 占用了簇 3, 5, 8, 9:

FAT[3] = 5      → 下一个簇是 5
FAT[5] = 8      → 下一个簇是 8
FAT[8] = 9      → 下一个簇是 9
FAT[9] = 0xFFFFFFF  → 这是最后一个簇（EOF）

文件内容 = 簇3的数据 || 簇5的数据 || 簇8的数据 || 簇9的数据
```

### 1.4 目录项结构 (32 字节)

每个目录项 32 字节，短文件名目录项格式：

| 偏移 | 大小 | 字段 | 说明 |
|------|------|------|------|
| 0x00 | 8 | `DIR_Name` | 文件名（不足补空格，小写转大写） |
| 0x08 | 3 | `DIR_Ext` | 扩展名 |
| 0x0B | 1 | `DIR_Attr` | 属性（0x01=只读, 0x02=隐藏, 0x10=子目录, 0x0F=长文件名） |
| 0x0C | 1 | DIR_NTRes | 保留 |
| 0x0D | 1 | DIR_CrtTimeTenth | 创建时间（毫秒） |
| 0x0E | 2 | DIR_CrtTime | 创建时间 |
| 0x10 | 2 | DIR_CrtDate | 创建日期 |
| 0x12 | 2 | DIR_LastAccessDate | 最后访问日期 |
| 0x14 | 2 | DIR_FstClusHI | **起始簇号高 16 位** |
| 0x16 | 2 | DIR_WrtTime | 最后写入时间 |
| 0x18 | 2 | DIR_WrtDate | 最后写入日期 |
| 0x1A | 2 | **DIR_FstClusLO** | **起始簇号低 16 位** |
| 0x1C | 4 | DIR_FileSize | 文件大小（字节） |

**起始簇号 = (DIR_FstClusHI << 16) | DIR_FstClusLO**

**长文件名目录项**：属性为 0x0F，每项存 13 个 Unicode 字符（UTF-16LE），多个连续项拼接成一个长文件名，**倒序排列**（最后一项在最前面）。

### 1.5 删除标记

当文件被删除（不是快速格式化）：
- 目录项的 `DIR_Name` 首字节被改为 **`0xE5`**
- FAT 表中该文件对应的簇链全部置 **`0x00000000`**（空闲）
- **数据区内容不变**

快速格式化则是：
- 清空引导扇区 BPB（可能重写）
- **清空整个 FAT 表**（全部置 0）
- **清空根目录区**
- **数据区完全不变**

---

## 2. 恢复原理

快速格式化后，我们有三个关键线索可以利用：

### 线索 1: FAT 表备份

FAT-32 通常有 2 份 FAT 表（主 + 备份）。快速格式化工具有时只清零主 FAT 表（FAT1），备份 FAT 表（FAT2）可能**完好无损**。

### 线索 2: 目录项残留

根目录被清空，但**子目录**的目录项在数据区的簇中，只要该簇未被覆盖，目录项信息（文件名、起始簇号、文件大小）就还在。

### 线索 3: 数据区完好

所有文件的实际内容仍在原簇中，只要知道文件的起始簇号和簇链，就能完整恢复。

---

## 3. 恢复步骤（OS 实验方案）

### Step 1: 读取并解析引导扇区

```c
// 读取 LBA 0
uint8_t boot[512];
read_sector(img_fd, 0, boot);

// 解析 BPB
uint16_t bytsPerSec = *(uint16_t*)&boot[0x0B];
uint8_t  secPerClus = boot[0x0D];
uint16_t rsvdSecCnt = *(uint16_t*)&boot[0x0E];
uint8_t  numFATs    = boot[0x10];
uint32_t fatSz32    = *(uint32_t*)&boot[0x24];
uint32_t rootClus   = *(uint32_t*)&boot[0x2C];

// 计算各区域起始
uint32_t fat1_start  = rsvdSecCnt;
uint32_t fat2_start  = fat1_start + fatSz32;
uint32_t data_start  = fat2_start + fatSz32;
```

### Step 2: 检查并尝试恢复 FAT 表

```c
// 先读主 FAT 表
uint8_t *fat1 = malloc(fatSz32 * bytsPerSec);
read_sectors(img_fd, fat1_start, fatSz32, fat1);

// 检查主 FAT 表是否为空
int fat1_empty = 1;
for (int i = 2; i < fatSz32 * bytsPerSec / 4; i++) {
    if (((uint32_t*)fat1)[i] != 0) { fat1_empty = 0; break; }
}

if (fat1_empty) {
    // 尝试从备份 FAT 表恢复
    uint8_t *fat2 = malloc(fatSz32 * bytsPerSec);
    read_sectors(img_fd, fat2_start, fatSz32, fat2);

    int fat2_valid = 0;
    // 验证备份 FAT 的合理性：检查 EOF 标记
    for (int i = 2; i < fatSz32 * bytsPerSec / 4; i++) {
        uint32_t val = ((uint32_t*)fat2)[i];
        if (val >= 0x0FFFFFF8 && val <= 0x0FFFFFFF) {
            fat2_valid = 1; break;
        }
    }

    if (fat2_valid) {
        // 将备份 FAT 复制到主 FAT 位置
        write_sectors(img_fd, fat1_start, fatSz32, fat2);
        printf("[恢复] 从备份 FAT 恢复成功\n");
        fat1 = fat2;
    } else {
        printf("[警告] FAT 表为空且无有效备份，需要扫描重建\n");
    }
}
```

### Step 3: 扫描目录项

如果 FAT 表和根目录都被清空，需要**扫描整个数据区**寻找目录项（0xE5 标记的已删除项或 `0x2E` 的目录项）：

```c
void scan_directory(uint8_t *data, uint32_t clus, uint32_t secPerClus,
                    uint32_t bytsPerSec, int depth) {
    uint32_t offset = (clus - 2) * secPerClus * bytsPerSec;
    uint8_t *clus_data = data + offset;

    for (int i = 0; i < secPerClus * bytsPerSec; i += 32) {
        uint8_t *entry = clus_data + i;
        uint8_t name_byte = entry[0];

        // 空目录项或结束标记
        if (name_byte == 0x00) break;         // 本簇目录结束
        if (name_byte == 0xE5) continue;       // 已删除的文件

        uint8_t attr = entry[0x0B];
        if (attr == 0x0F) continue;            // 长文件名项，跳过

        // 提取文件名（8+3 格式）
        char name[13];
        snprintf(name, 9, "%8s", entry);
        snprintf(name+8, 4, "%3s", entry+8);
        // 去掉尾部空格
        rtrim(name);

        uint16_t clus_hi = *(uint16_t*)&entry[0x14];
        uint16_t clus_lo = *(uint16_t*)&entry[0x1A];
        uint32_t file_clus = (clus_hi << 16) | clus_lo;
        uint32_t file_size = *(uint32_t*)&entry[0x1C];

        printf("[%s] %s 起始簇=%u 大小=%u\n",
               (attr & 0x10) ? "目录" : "文件", name, file_clus, file_size);

        // 如果是子目录，递归扫描
        if ((attr & 0x10) && file_clus >= 2) {
            scan_directory(data, file_clus, secPerClus, bytsPerSec, depth+1);
        }
    }
}
```

### Step 4: 追踪簇链重建文件

从目录项拿到起始簇号后，通过 FAT 表追踪整个簇链：

```c
void read_file_clusters(uint8_t *data, uint32_t *fat_table,
                        uint32_t start_clus, uint32_t secPerClus,
                        uint32_t bytsPerSec, uint8_t *output,
                        uint32_t *out_size) {
    uint32_t clus = start_clus;
    uint32_t offset = 0;

    while (clus >= 2 && clus < 0x0FFFFFF8) {
        uint32_t lba = data_start + (clus - 2) * secPerClus;
        uint32_t clus_offset = (clus - 2) * secPerClus * bytsPerSec;

        memcpy(output + offset, data + clus_offset, secPerClus * bytsPerSec);
        offset += secPerClus * bytsPerSec;

        clus = fat_table[clus];  // FAT 表项指向下一个簇
    }

    *out_size = offset;
}
```

### Step 5: 重建 FAT 表（如果没有备份）

如果主 FAT 表和备份 FAT 都被清零了，需要扫描数据区来**推断簇链**：

思路：遍历数据区的每个簇，检查簇开头是否有文件头签名（magic number），然后根据文件大小推断其占用的簇数。但这种方法无法处理碎片化文件，只能恢复连续存储的文件。

更实用的替代方案：**使用 PhotoRec 等工具的文件雕刻（File Carving）技术**，按文件尾部签名来识别文件边界。

```
每个簇扫描:
  ├─ 检查 magic number → 确定文件类型
  ├─ 连续读取后续簇直到遇到下一个 magic 或文件尾部签名
  └─ 重建 FAT 链: FAT[n] = n+1, FAT[n+k] = 0xFFFFFFF
```

---

## 4. 实验恢复流程总结

```
快速格式化后的磁盘镜像
        │
        ▼
  Step 1: 读取 BPB，计算各区域偏移
        │
        ▼
  Step 2: 检查 FAT1 → 是否全零？
        ├── 否 → 直接使用
        └── 是 → 检查 FAT2（备份）→ 复制到 FAT1
                    └── FAT2 也无效 → 标记为"需要扫描重建"
        │
        ▼
  Step 3: 扫描数据区目录项（0x2E 标记）
        │   → 找到所有文件和子目录的起始簇号
        ▼
  Step 4: 用 FAT 表追踪簇链 → 提取文件数据
        │
        ▼
  Step 5: 按目录结构重建文件系统（或者导出到宿主机）
```

## 5. 调试技巧

- **用 hexdump 查看原始数据**：`xxd -l 512 disk.img` 查看引导扇区
- **检查 FAT 表**：`xxd -s $((rsvdSecCnt * 512)) -l $((fatSz32 * 512)) disk.img`
- **验证恢复**：用 `file` 命令检查恢复的文件类型是否正确
- **常见陷阱**：
  - 忘记 BPB 中字段可能是小端序
  - 根目录也是一个普通簇（由 `BPB_RootClus` 指定），不是固定位置
  - 簇号从 2 开始计数（簇 0 和 1 有特殊含义）
  - 长文件名项倒序排列，需要反向拼接

# FAT
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260612153746.png)
如何遍历 FAT Region?


Read 4.1 to know:
Given any valid cluster number N, this section describes the algorithm used to determine the entry in the FAT (s) for that cluster number.

这里的找到 cluster 即数据 sector 的开始，公式：
$$
\text{SectorNumber}=(\text{FatNumber}*\text{FATSz}) +\text{ThisFATSecNum}
$$

$ThisFATSecNum$ is the sector number of the FAT sector that contains the entry for cluster N in the first FAT.
> The first two entries in the FAT are reserved. (*Section* 4.2)


# What need you do?
## 分类问题
root cluster 和 fat 都 gone 了，但是数据区永存。你可以尝试找到分类目录和数据。


在你对 FAT32 文件系统有了足够的认识以后，你会发现在我们的问题中，数据区的 clusters 分成以下几种情况：

1. 目录文件，存储若干目录项 (directory entry)，对应手册 Sections 6 和 7 描述的内容。注意 Section 7 是非常重要的，因为你必须恢复出完整的文件名；
2. BMP 文件的头部，以 “`424d` (`BM`)” 开头；
3. BMP 文件的实际数据，按照 3 字节一组按顺序存储了图片中的所有像素；
4. 未使用的 cluster。

在 FAT 表被清除后，我们已经无法根据 FAT 恢复出目录的树状结构了。因此接下来我们要做的是一个分类问题：我们需要依次扫描磁盘中的所有 clusters，并将它们标记为以上 4 类。你不需要使用任何机器学习——你可以手工硬编码一些特征，就足够完成识别了，例如目录文件里总是包含大量的 “BMP” 字符，这是数据和 bitmap 文件头所没有的。你不需要做得 100% 准确，因为你只要恢复相当一部分文件即可；但你要小心地编写健壮的代码，使得分类错误发生时程序不会发生太大的问题——你的程序可能在分类错误时 (例如将位图数据解析为目录时) 因为非法的输入而 crash。 


## 恢复文件名
这里着重阅读 section 7
我们要处理长文件名。

> [!Note] Long file names
> 存储是 reverse order 的 (Last entry in the set is stored first, followed by entry n-1, followed by entry n-2...)

Long file name （对于一个目标文件，或一个子目录）。实现方式是 a set of additional directory entries associated with the short name directory entry.
- They must immediately precede the corresponding short name directory and is, therefore, physically contiguous with the short name directory.

这里要注意一下，字符串的截断标志为 `0x0000`，而一个 `char` 是一字节 `0xff` 这么宽。所以我们需要对 `u8` 这个 name 的数组两个两个字节的看，用 `ASCII` 解释。

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260613213414.png)


```
扫描目录 dent
    │
    ├─ 找到 LAST_LONG_ENTRY (Ord & 0x40)
    │   ├─ 解析长文件名 ← 已有
    │   └─ 拿到短条目 (dent[i + seq_len])
    │       ├─ start_clus = (DIR_FstClusHI << 16) | DIR_FstClusLO
    │       ├─ file_size = DIR_FileSize
    │       └─ 如果是 BMP 文件 → 从 start_clus 开始连续写入
    │
    └─ 找到普通短条目 (Attr 不是 0x0F、不是目录)
        ├─ 用 format_83_name 拿到 8.3 文件名
        └─ 同上，连续写入
```

### **如何排查与修正代码？**

如果你正在编写恢复算法，可以尝试从以下几个技术点切入优化：

1. **检查 `bfOffBits`（像素数据偏移量）**： 确保你的代码正确解析了 BMP 头部（Header）中的 `bfOffBits`（通常在文件开头的第 10-13 字节）。如果头部本身被破坏或偏移量算错，整个图像的像素色彩和对齐都会错乱。但鉴于你的图下半部分极其完美，头部数据可能已经被你通过某种方式补全或恢复了，问题主要在数据区。
    
2. **寻找真正的文件起始簇（Cluster Chain Reconstruction）**： 因为 FAT 表丢失，如果该文件不是百分之百连续存储的，顺序读取（Raw carving）就会把无用的碎片读进来。
    
    - **解决思路**：如果能找到该 BMP 图像的完整文件头（以 `BM` 开头，即 `0x42 0x4D`），利用其中的 `bfSize` 获取文件总大小。
        
    - 如果该图非常重要，需要利用 **智能雕刻（Smart Carving）** 算法，通过验证 BMP 每一行像素的连续性（即边缘梯度检测），剔除掉中间插进来的那段“彩色条纹”垃圾碎片，将真正属于 BMP 的簇拼接起来。
        
3. **方向颠倒特性（倒序存储）**： 值得注意的是，标准的 Windows 位图（BMP）在内存中是**自下而上（Bottom-Up）**存储的。也就是说，文件数据区里的第一行字节，对应的是图片的**最底端**。
    
    - 对应你的结果：图片下半部分正常，说明**文件数据区的开端（前半部分）是错误的**（对应图片上方）；而**文件数据区的末尾（后半部分）是正确的**（对应图片下方）。这也再次印证了是文件的首部簇被其他数据覆盖或发生了断裂。

## 一、核心思路：双重验证（Validator）

在循环寻找下一个簇时，不能盲目 `c++`。每次决定写入下一个簇前，必须通过两关：

1. **类型初筛**：利用你现有的 `gl_cluses[i].type == CLUS_BMP_DATA`。
    
2. **像素梯度连续性验证（Edge Gradient Check）**：这是 Smart Carving 的精髓。BMP 图像是按行排列像素的。**前一个簇的最后一行像素**，和**备选簇的第一行像素**，在物理世界中通常是连续的（渐变的）。如果突然断层（比如变成全是 0 或者是无序的系统代码），两行像素之间的颜色差值（梯度）会发生惊人的暴增。
    

## 二、 具体改造方案

### 1. 提取 BMP 关键元数据

在 `recover_bmp_file` 刚拿到第一个簇（Header）时，你需要解析出 BMP 的**像素开始偏移量**、**图片宽度**和**色彩位数（BPP）**。

C

```
// 在 recover_bmp_file 内部解析
u32 bfOffBits = *(u32*)(&cl->data->raw[10]); // 像素数据从哪里开始
u32 biWidth = *(u32*)(&cl->data->raw[18]);   // 宽度（像素）
u16 biBitCount = *(u16*)(&cl->data->raw[28]);// 24位（RGB）还是32位（RGBA）

// 计算每一行像素实际占用的字节数（BMP要求4字节对齐！）
u32 row_size = ((biWidth * biBitCount + 31) / 32) * 4;
```

### 2. 编写像素连续性验证函数

我们需要一个函数来评估：**如果把 `next_cl` 接在 `curr_cl` 后面，视觉上合不合理？**

这里可以通过计算两行对应像素的 **欧几里得距离（或绝对差值和 SAD）**。如果平均差值超过某个阈值（比如 50），说明这个簇是错误的碎片。

C

```
// 这是一个简化版的边缘匹配评估函数
double calculate_row_difference(const u8* last_row, const u8* next_row, u32 row_size) {
    double total_diff = 0;
    for (u32 i = 0; i < row_size; i++) {
        total_diff += abs((int)last_row[i] - (int)next_row[i]);
    }
    return total_diff / row_size; // 返回平均每个字节的差异
}
```

### 3. 重构你的主循环：从 `for` 变成 `While + 动态搜索`

你需要抛弃 `c < first_clus + num_clus` 这种僵化的自增，引入一个**寻簇指针**。

C

```cpp
u32 clus_needed = num_clus;
u32 current_clus_id = first_clus;
u32 written_clusters = 0;

// 先写入第一个簇（Header 簇）
cluster* curr_data = cluster_to_sec(current_clus_id, hdr);
fwrite(curr_data->raw, 1, CLUS_BYTE_COUNT, fp);
written_clusters++;

while (written_clusters < clus_needed) {
    u32 best_next_clus = 0;
    double min_diff = 1e9; // 找一个差异最小的簇值
    
    // 假设当前簇已经进入了像素区（written_clusters 足够大，越过了 bfOffBits）
    // 拿到当前写入文件的最后 row_size 字节（即最后一行像素）
    // 提示：你需要根据当前写了多少字节计算最后一行像素在 curr_data->raw 中的偏移
    u8* last_row_ptr = &curr_data->raw[CLUS_BYTE_COUNT - row_size]; 

    // 搜索整个磁盘中所有标记为 CLUS_BMP_DATA 的空闲簇/潜在簇
    // 为了提高效率，可以先从 current_clus_id + 1 开始向后局部搜索
    for (u32 cand = 2; cand < gl_total + 2; cand++) {
        if (gl_cluses[cand - 2].type != CLUS_BMP_DATA) continue;
        if (cand == current_clus_id) continue; // 跳过自身

        cluster* cand_data = gl_cluses[cand - 2].data;
        u8* first_row_ptr = cand_data->raw; // 备选簇的第一行像素

        // 计算连续性得分
        double diff = calculate_row_difference(last_row_ptr, first_row_ptr, row_size);
        
        if (diff < min_diff) {
            min_diff = diff;
            best_next_clus = cand;
        }
    }

    // 设置一个阈值，如果全盘最匹配的簇都极其违和，说明可能遇到了损坏或到文件尾了
    if (best_next_clus == 0 || min_diff > 80.0) { 
        LOG("无法找到匹配的连续像素簇，雕刻中断");
        break;
    }

    // 找到了真正接得上的下一个簇！
    LOG("Smart Carve: 跨越断层，从簇 %u 跳入真正连续的簇 %u (Diff: %.2f)", current_clus_id, best_next_clus, min_diff);
    
    current_clus_id = best_next_clus;
    curr_data = cluster_to_sec(current_clus_id, hdr);
    
    size_t to_write = (written_clusters == clus_needed - 1) ? (bmp_sz % CLUS_BYTE_COUNT) : CLUS_BYTE_COUNT;
    if (to_write == 0) to_write = CLUS_BYTE_COUNT;

    fwrite(curr_data->raw, 1, to_write, fp);
    
    // 为了防止这个簇被重复复用，可以将其状态改写
    gl_cluses[current_clus_id - 2].type = CLUS_FREE; 
    
    written_clusters++;
}
```

## 三、 针对 BMP 特性的两点致命细节（避坑指南）

1. **BMP 的倒序（Bottom-Up）特性**： 在上一问提到了，BMP 的像素在磁盘里是**从图片的最后一行往第一行反向存储的**。
    
    - **这意味着什么？** 你的代码是从文件头往后读的，也就是说，在你的 `for` 循环里，你是**先读到图片的最底端像素，最后读到图片的最顶端像素**。
        
    - **对调边缘：** 当你验证 `curr_cl` 和 `next_cl` 是否连续时，实际上是在验证**图像中某一行与其上方一行**的连续性。在逻辑上这完全成立（因为上下行像素依然是渐变的），你的算法不需要颠倒读取顺序，只需要保持“前一个簇的最后几字节”与“后一个簇的头几字节”进行对比即可。
        
2. **跳过 BMP Header 簇的干扰**： 文件的前几个簇（取决于图片大小和头部大小）包含的是文件头信息（`BITMAPFILEHEADER` 和 `BITMAPINFOHEADER`），甚至可能包含调色板（Palette）。
    
    - 在 `written_clusters` 很少、还没有跨越 `bfOffBits` 偏移量时，**不要进行像素梯度校验**，因为那时候的数据是结构体而不是像素。在前几个簇，你依然可以采用 `c++` 顺序读取，直到写入的总字节数超过了 `bfOffBits`，再开启上面的智能 `SAD 梯度搜索`。