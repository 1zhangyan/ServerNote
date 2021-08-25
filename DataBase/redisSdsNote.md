# SDS

## SDS数据结构

### Redis 2.9实现
```C
typedef char *sds;
struct sdshdr {
    unsigned int len;
    unsigned int free;
    char buf[];
};
```
C的字符串以'\0'结尾，基于字符串的操作大多数都需要对'\0'进行判断，即使是判断字符串的长度，也是需要从头遍历进而计算字符串的实际长度。   
sds中结构与之类似，它的做法是根据字符串的大小申请一组连续的空间，从表面上看也是一个字符数组，其提升性能的关键额外信息是在这个字符数组的基础上，又申请一个header。  
header中记录了sds的长度，sds可用的存储空间，以及指向具体的字符串内容的指针。  
理论上说，对于记录了长度和大小的sds来说，C特征的字符串以'\0'结尾，这个特征完全可以不保留，但是为了兼容C的很多函数，sds保留了这一个字符在末尾，但是结束符并不会被记录在sds的长度len中。  

这种方式既保留了传统C处理字符串的特点，又解决了容量，越界，溢出等C字符串常见问题。但是，可以看见的是，sdshdr使用记录长度的字段 len 和 free 都是 unsinged int 类型，也就是说，无论存储的字符串大小为多少，长度字段将会一直是32位的。对于小字符串来说，这一定程度造成了内存的浪费。而Redis作为一个内存数据库，对于内存是非常敏感的，所以演进出了后面的几种不同类型的sds。   
***两个概念***  
- 空间预分配：提前分配多余的空间，字符串扩容的时候不必再申请空间。因为sds申请需要连续的空间，如果不额外申请，这次不够，扩容的话就会新申请一块大内存，然后把原来的字符串复制进去。  
- 惰性释放：释放的过程不会把多余的空间释放，而是仅仅改变 hdr 中 len 的数值，这样下次扩容的时候不需要再次申请空间。  

### Redis 3.2实现

```C
struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags; /* 低三位表示类型, 高五位表示字符串长度 */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* 字符串长度*/
    uint8_t alloc; /* 分配长度 */
    unsigned char flags; /* 低三位表示类型，高五位未使用 */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* 字符串长度*/
    uint16_t alloc; /* 分配长度 */
    unsigned char flags; /* 低三位表示类型，高五位未使用 */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* 字符串长度*/
    uint32_t alloc; /* 分配长度 */
    unsigned char flags; /* 低三位表示类型，高五位未使用 */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* 字符串长度*/
    uint64_t alloc; /* 分配长度 */
    unsigned char flags; /* 低三位表示类型，高五位未使用 */
    char buf[];
};
```
***注：***  
- 3.2 的源码和 2.9 不同，实现了对sds更加精细化的控制。可以看见的是，如果字符串长度较小的话，支持使用较小空间的 sdshdr 存储此种类型的字符串。
- flag 字段是为了给内部函数使用判断具体的 sds 类型。由于所有的 sds 都是以 char* 指针传参，这样内部的函数会根据 flag 字段就可以判断具体的 sds 类型。
- \__attribute__ ((\__packed__))是禁用了编译器的对齐优化，有的博客将其理解为节省内存是片面的，其实是后面的很多操作都是直接进行指针加减，使用指针直接指向特定含义的内存，从该处取数据，这就需要对内存有绝对的掌控。一旦编译器有优化，可能会导致指针指向的位置出现偏差。
