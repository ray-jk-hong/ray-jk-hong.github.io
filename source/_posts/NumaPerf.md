---
title: Numa perf
categories: 
- Linux MM
tags:
- Linux MM
---

```c
#include <stdio.h>
#include <stdbool.h>
#include <errno.h>
#include <string.h>
#include <sys/ioctl.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/time.h>
#include <sys/mman.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/syscall.h>

static int nr_numa;

static int open_device(const char *file)
{
	int fd = open(file, O_RDWR);
	if (fd < 0) {
		printf("open file /dev/sdma failed, err: %s\n", strerror(errno));
		exit(1);
	}

	return fd;
}

static bool huge_page = false;
enum {
        MPOL_DEFAULT,
        MPOL_PREFERRED,
        MPOL_BIND,
        MPOL_INTERLEAVE,
        MPOL_LOCAL,
        MPOL_MAX,       /* always last member of enum */
};
#define MPOL_MF_MOVE    (1 << 1)

#define SIZE_1M 0x100000UL
#define SIZE_2M 0x200000UL

static int testcase(size_t size, int fd, int src_nid, int dst_nid)
{
	int ret;
	long memcpy_dur;
	char *src, *dst;
	struct timeval start, end;
	size_t alloc_size = (size + SIZE_2M - 1UL) & ~(SIZE_2M - 1UL); // align up to 2M

	int flags = MAP_PRIVATE | MAP_ANONYMOUS;
	if (huge_page)
		flags |= MAP_HUGETLB;

	src = mmap(NULL, alloc_size, PROT_WRITE | PROT_READ, flags, -1, 0);
	if (src == MAP_FAILED) {
		printf("alloc src memory failed, %d\n", errno);
		return -1;
	}

	dst = mmap(NULL, alloc_size, PROT_WRITE | PROT_READ, flags, -1, 0);
	if (dst == MAP_FAILED) {
		printf("alloc dst memory failed, %d\n", errno);
		return -1;
	}

	unsigned long nodemask = 1UL << src_nid;
	ret = syscall(__NR_mbind, src, alloc_size, MPOL_BIND, &nodemask, nr_numa, MPOL_MF_MOVE);
	if (ret < 0) {
		printf("mbind for src failed\n");
		return -1;
	}

	nodemask = 1UL << dst_nid;
	ret = syscall(__NR_mbind, dst, alloc_size, MPOL_BIND, &nodemask, nr_numa, MPOL_MF_MOVE);
	if (ret < 0) {
		printf("mbind for dst failed\n");
		return -1;
	}

	memset(src, 'a', size);
	memset(dst, 'b', size);

	gettimeofday(&start, NULL);
	memcpy(dst, src, size);
	gettimeofday(&end, NULL);

	memcpy_dur = 1000000 * (end.tv_sec - start.tv_sec) + end.tv_usec - start.tv_usec;
    
	printf("src: %3d, dst: %3d, size: %8zd, memcpy: %12ld, %s\n",
		src_nid, dst_nid, size, memcpy_dur, huge_page ? "huge_page" : "normal_page");

	return 0;
}

int main(int argc, char **argv)
{
	int ret, pasid;
	int src_nid, dst_nid;

	if (argc < 6) {
		printf("invalid input!\n"
			"Usage:\n\t%s <dev_file> <size> <src_nid> <dst_nid> <nr_numa> [huge_page]\n",
			argv[0]);
		return -1;
	}

	if (argc == 7)
		huge_page = true;

	int fd = open_device(argv[1]);
	if (fd < 0) {
		perror(argv[1]);
		exit(1);
	}
	size_t size = atoi(argv[2]);
	src_nid = atoi(argv[3]);
	dst_nid = atoi(argv[4]);
	nr_numa = atoi(argv[5]);

	testcase(size, fd, src_nid, dst_nid);

	close(fd);

	return 0;
}

```
