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
