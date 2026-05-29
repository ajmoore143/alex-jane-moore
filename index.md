# Всем привет! Это мой сайт

*~Code~*
```
#include <stdlib.h>
#include <stdio.h>
#include <stdint.h>
#include <sys/mman.h>
// uint64_t == 8 byte

#define HEAP_SIZE 400
#define CHUNK_SIZE 8

uint64_t *HEAP_START = NULL;

void init_heap() {
    // Hey OS! where can I start my heap?
    uint64_t * heap = mmap(NULL, HEAP_SIZE, PROT_READ | PROT_WRITE, MAP_ANON | MAP_SHARED, -1, 0);
    HEAP_START = heap;
    // more setup?
    *HEAP_START = HEAP_SIZE; // writing header
}

void *my_malloc(size_t size) {
    uint64_t *current = HEAP_START;
    while (current < (HEAP_START + (HEAP_SIZE / CHUNK_SIZE))) {
        // Traverse the heap
        uint64_t cur_header = *current; // reading header
        uint64_t cur_size = cur_header & (~1); // get size
        uint64_t is_free = ~cur_header & 1;
        if (is_free && (size + sizeof(uint64_t) <= cur_size)) {
            // Proceed
            // Split the block into malloc'd and free parts
            // if needed
            // Make sure alloc'd block is multiple of 8
            // Round up to the next multiple of 8
            size_t size_w_hdr = size + sizeof(uint64_t);
            size_t rounded = ((size_w_hdr + 7) / 8) * 8;
            // check if we need to split or not
            if (cur_size > rounded + 8) { // +8 so no dangling 8-byte chunks
                *current = rounded + 1; // set alloc'd bit
                // split
                uint64_t *remaining = current + (*current / CHUNK_SIZE);
                *remaining = cur_size - rounded;
            } else {
                *current += 1;
            }
            return current + 1;
        } else {
            // not in a free block
            // go to my next block
            uint64_t *next = current + (cur_size / CHUNK_SIZE);
            current = next;
        }
    }
    return NULL;
}

void my_free(void *p) {
    uint64_t *current = p;
    uint64_t *header = current - 1;
    if (*header & 1) {
        *header = *header & ~1;
    }
}

void print_heap() {
    uint64_t *current = HEAP_START;
    while (current < (HEAP_START + (HEAP_SIZE / CHUNK_SIZE))) {
        uint64_t cur_header = *current;
        uint64_t cur_size = (cur_header / 2) * 2;
        printf("%p\t%ld\t%ld\n", current, cur_header % 2, cur_size);
        uint64_t *next = current + (cur_size / CHUNK_SIZE);
        current = next;
    }
    printf("\n\n");
}

int main() {
    init_heap();

    print_heap();
    int *a = my_malloc(40);
    print_heap();
    int *b = my_malloc(10);
    print_heap();
    my_free(a);
    print_heap();
    int *c = my_malloc(20);
    print_heap();
    my_free(b);
    my_free(c);
}
```
