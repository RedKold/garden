# `kvdb_open`

要实现这个 api，我们要能够从一个以内存形式存储了的文件中恢复数据库。

用 `open` 打开文件之后，就可以开始将它和我们的 `struct kvdb_t` 绑定了

```c
int kvdb_open(struct kvdb_t *db, const char *path) {
    memset(&db->buffer, 0, sizeof(db->buffer));
    // if the path exist, then we load it

    int fd = open(path, O_RDWR | O_CREAT, 0666);
    if (fd == -1)
        return -1;

    char *dup_path = strdup(path);
    if (!dup_path) {
        close(fd);
        return -1;
    }

    void *buf = malloc(8192);
    if (!buf) {
        close(fd);
        free(dup_path);
        return -1;
    }

    db->fd = fd;
    db->path = dup_path;
    db->buffer.data = buf;
    db->buffer.capacity = 8192;
    db->buffer.size = 0;
    db->file_size = 0;

    // Load existing data from file into buffer
	// measure the size of file
    off_t sz = lseek(fd, 0, SEEK_END);
    if (sz > 0) {
        if ((size_t)sz > db->buffer.capacity) {
            // Grow buffer to fit the file
            void *newbuf = realloc(db->buffer.data, sz);
            if (!newbuf) {
                close(fd);
                free(buf);
                free(dup_path);
                return -1;
            }
            db->buffer.data = newbuf;
            db->buffer.capacity = sz;
        }
		// move the file pointer to the beginningc
        lseek(fd, 0, SEEK_SET);
        ssize_t n = read(fd, db->buffer.data, sz);
        if (n != sz) {
            close(fd);
            free(db->buffer.data);
            free(dup_path);
            return -1;
        }
        db->buffer.size = sz;
        db->file_size = sz;
    }

    return 0;
}
```

# `kvdb_put`
All you need to know is `Append only` can form any data structure.

我们并不需要像一个传统的*表格*一样，只需要不断的 append 就行了。我们查找时候，只要找到最新的数据即可。
```c
int kvdb_put(struct kvdb_t *db, const char *key, const char *value) {
    u32 key_len = strlen(key);
    u32 val_len = strlen(value);
    u32 total_size = key_len + val_len + sizeof(key_len) + sizeof(val_len);

    // Pack entry: [key_len][val_len][key_data][val_data]
    char buf[total_size];
    char *p = buf;
    memcpy(p, &key_len, sizeof(key_len)); p += sizeof(key_len);
    memcpy(p, &val_len, sizeof(val_len)); p += sizeof(val_len);
    memcpy(p, key, key_len);             p += key_len;
    memcpy(p, value, val_len);

    return append_write(db, buf, total_size);
}
```

深入理解一个事实
如果实现 *Append only*，就不需要原地对表修改
只要执行最末修改即可。


> [!Note] 著名持久化数据结构等式
> **Random read + append-only write = 持久化数据结构**


Append-only 也天然适配我们在缓存里写 WAL。

这里我们的 `int append_write(struct kvdb_t *db, const char* data, size_t size)` 这样实现：

如果新数据超出 buffer 容量，则 `flush_buffer`，并重置 buffer 的大小。然后把新数据复制到 `data+size` 指针位置。同时维护 `buffer` 的 size

```c
static int append_write(struct kvdb_t *db, const char *data, size_t size) {
    if (!db->buffer.data || size > db->buffer.capacity)
        return -1;

    // Flush if the new data won't fit.
    if (size + db->buffer.size > db->buffer.capacity) {
        if (flush_buffer(db) != 0)
            return -1;
        db->buffer.size = 0;
        db->file_size = 0;
    }

    memcpy(db->buffer.data + db->buffer.size, data, size);
    db->buffer.size += size;
    return 0;
}
```


# `kvdb_get`
这里实现“查”

只要找到数据库中的 `_key` 和要找的 key 匹配的，然后把 `v` 返还即可。

```c
int kvdb_get(struct kvdb_t *db, const char *key, char *buf, size_t length) {
    struct kvdb_iter it;
    kvdb_iter_init(&it, db);

    size_t search_len = strlen(key);
    const char *found_val = NULL;
    u32 found_val_len = 0;

    const char *k, *v;
    u32 klen, vlen;
    while (kvdb_iter_next(&it, &k, &v, &klen, &vlen)) {
        if (klen == search_len && memcmp(k, key, search_len) == 0) {
            found_val = v;
            found_val_len = vlen;
        }
    }

    // cannot find, return -1;
    if (!found_val)
        return -1;

    size_t to_copy = found_val_len < length - 1 ? found_val_len : length - 1;
    if(buf){
        memcpy(buf, found_val, to_copy);
        buf[to_copy] = '\0';
    }

    return to_copy;
}
```

这个是 `O(n)` 的查找，当然很慢，但是这个实验我们先不关心效率了。实际的数据库实现应该用红黑树等技术。


# `kvdb_close`

直接附上代码吧

```c
int kvdb_close(struct kvdb_t *db) {
    int ret = flush_buffer(db);
    if (db->fd >= 0)
        close(db->fd);
    free(db->buffer.data);
    free((void *)db->path);
    memset(db, 0, sizeof(*db));
    return ret;
}
```


# 更优雅的遍历数据库
我们提供了一个 `kvdb_iter` 的操作

```c
struct kvdb_iter {
    const char *cur;
    const char *end;
};

```

```c
void kvdb_iter_init(struct kvdb_iter *iter, const struct kvdb_t *db) {
    iter->cur = db->buffer.data;
    iter->end = db->buffer.data + db->buffer.size;
}

int kvdb_iter_next(struct kvdb_iter *iter,
                   const char **key, const char **value,
                   u32 *key_len, u32 *val_len) {
    if (iter->cur >= iter->end)
        return 0;

    *key_len = *(const u32 *)iter->cur;
    *val_len = *(const u32 *)(iter->cur + sizeof(u32));
    *key = iter->cur + sizeof(u32) + sizeof(u32);
    *value = *key + *key_len;

    iter->cur += sizeof(u32) + sizeof(u32) + *key_len + *val_len;
    return 1;
}
```






