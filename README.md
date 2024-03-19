# cachetop

> Author: Xin Shuichen<zhangyuchen.prog@foxmail.com>

## Description

`cachetop` is a program that displays the top N files occupying the most cache in the specified path (default is the root directory '/'). It provides insights into cache usage by individual files.

## Advantages && Principle

- **Low memory usage**: The program consumes minimal memory, not exceeding 20M RSS on the author's machine, regardless of any metric changes.
- **Cache coverage**: It displays page cache even if it is not associated with a process.
- **Selective file system support**: The program excludes all virtual file systems and currently only reads ext4 and tmpfs file systems.

### Principle and Mechanism
The core principle of this program is based on the mincore system call, which is inspired by `hcache` [https://github.com/djhuahao/hcache].
The mincore system call allows checking whether each page in a memory segment is cached or not. To utilize this, the file needs to be mapped using mmap, and then the mincore is called with the mmap's address space to obtain the cache information.

### Reasons for Not Using Existing Tools
The reason for not directly using tools like `hcache` or `vmtouch`, which are based on `pcstat` [https://github.com/tobert/pcstat], is that these tools obtain the filenames that need cache information from /proc/pid/maps and /proc/pid/fd.

In Linux kernel, if a process opens a file, reads/writes to it, and then closes the file, all the caches of this file will not be reclaimed even if no one else is using the file. These caches become orphaned and cannot be found under the /proc directory. Therefore, the cache captured by tools like hcache and vmtouch may differ significantly from the cache shown by the free tool.

To obtain all the cache data, it is necessary to traverse all system files and capture cache data for each file. During the capturing process, virtual file systems like procfs and sysfs do not need to be queried. This program uses a whitelist approach, and currently, it only queries data under ext4 and tmpfs mount points by default.

### Optimizations

The following optimizations have been made to achieve O(1) space complexity:

1. The core scenario for using cache-like tools is to see the top n caches, and it is unnecessary to cache all the data. Therefore, a min-heap is used to complete the top n selection. The size of the top n records in memory remains constant, regardless of the number of files.
2. The vec is read in segments, with a maximum of 10G per read, corresponding to approximately 2M memory (10G / 4096 = 2560KB). This way, the memory consumption is not related to the file size, and the memory usage of the entire program is fixed.

The following optimizations have been made to reduce unnecessary reads:
1. For files that are already smaller than the current top n cache size, they are directly skipped because their cache cannot be larger than the top n files. During testing, this optimization reduced unnecessary mincore operations by an average of **97%**.

### Considerations and Performance:

It is important to note that because this program reads all files, it is impossible to avoid the growth of dentry and inode. However, these are reclaimable slab objects and will not have a significant impact.

On the author's PC, after dropping the cache, reading all **710,000** files takes approximately **20** seconds (`6.37s user 11.12s system 98% cpu 17.694 total`), with the actual number of mincore operations being **20,000**.

The dentry and inode cache grew by **1G** during this collection. This should have a linear relationship with the number of files.

Using the command `watch -n 1 "cat /proc/$(ps -ef | grep cachetop | grep -v grep | awk '{print $2}')/status | grep RSS"`, it can be observed that the memory usage never exceeds **15M**.

Test Summary:
```
File scanned: 710,000 (depend on env)
Mincore executed: 20,000 (depend on env)
Time Spent: 17.694s (depend on scan file num)
Memory Max: 15MB (fixed value)
Dentry & Inode Cache Grew: 1G (depend on scan file num)
```

## Usage

```
usage: cachetop [-h] [--topn TOPN] [--include_fs INCLUDE_FS] [--exclude_dir EXCLUDE_DIR] [search_path]

positional arguments:
	search_path									The directory path to search for files (default: '/')

optional arguments:
	-h, --help									show this help message and exit
	--topn TOPN, -n topn						Print the top N files occupying the most cache (default: 10)
	--include_fs INCLUDE_FS, -f INCLUDE_FS		Comma-separated list of file systems to include (default: 'ext4,tmpfs')
	--exclude_dir EXCLUDE_DIR, -e EXCLUDE_DIR	Pipe-separated list of directories to exclude (e.g., '/test|/test1')
```

## Example

1. Search all files under the root directory and print the top 5 files occupying the most cache:
```
cachetop -n 5
```

2. Search all files under the /test directory and print the top 5 files occupying the most cache:
```
cachetop /test -n 5
```

3. Search all files under the root directory, print the top 10 files, and include nfs, bcachefs, and tmpfs file systems:
```
cachetop -n 5 --include_fs "nfs,bcachefs,tmpfs"
```

4. Search files under the root directory, excluding files in the /root and /home directories:
```
cachetop --exclude_dir "/root|/home"
```

## Example Result

```shell
$ time ./cachetop -n 20               
FILE NAME                                                                                                                           | CACHE SIZE           | FILE SIZE           
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
/usr/lib/sysimage/rpm/rpmdb.sqlite                                                                                                  | 170.86 MB            | 174.05 MB           
/var/cache/dnf/fedora-filenames.solvx                                                                                               | 53.64 MB             | 53.66 MB            
/var/cache/dnf/fedora-376ef8e983c65ce0/repodata/5f86dfe1903316d6b25d494616cf18352204fe70c529b4c97859df6faecd493f-filelists.xml.zck  | 49.40 MB             | 49.40 MB            
/usr/lib64/libclang-cpp.so.16                                                                                                       | 45.96 MB             | 66.50 MB            
/usr/lib64/libLLVM-16.so                                                                                                            | 42.55 MB             | 117.79 MB           
/var/cache/dnf/updates-filenames.solvx                                                                                              | 34.77 MB             | 34.77 MB            
/var/cache/dnf/fedora-376ef8e983c65ce0/repodata/dfa2aec8d2e83459677d698542f1a190d92815888668d027b21dfd46fb86ce01-primary.xml.zck    | 31.68 MB             | 31.68 MB            
/var/log/journal/4483974d999a4003bae5281e394062bb/system.journal                                                                    | 31.02 MB             | 32.00 MB            
/var/cache/dnf/updates-b7ba662710b98f1a/repodata/f726172e0c8e64f0d991172a3c5945a24cf8d79d315e47e27688f5bc01511b24-filelists.xml.zck | 30.09 MB             | 30.09 MB            
/root/anaconda3/libexec/gcc/x86_64-conda-linux-gnu/11.2.0/cc1                                                                       | 29.62 MB             | 29.62 MB            
/root/anaconda3/pkgs/gcc_impl_linux-64-11.2.0-h1234567_1/libexec/gcc/x86_64-conda-linux-gnu/11.2.0/cc1                              | 29.62 MB             | 29.62 MB            
/usr/lib/locale/locale-archive                                                                                                      | 25.98 MB             | 213.97 MB           
/usr/lib/locale/locale-archive.real                                                                                                 | 25.98 MB             | 213.97 MB           
/var/cache/dnf/fedora.solv                                                                                                          | 25.49 MB             | 25.99 MB            
/var/lib/docker/overlay2/469328597deff3ff11e139b1b279dd15f802ec66c27984be99877e833bec407f/diff/coze-discord-proxy                   | 18.50 MB             | 28.84 MB            
/root/jellyfin/config/data/library.db                                                                                               | 17.88 MB             | 17.88 MB            
/var/cache/dnf/updates-updateinfo.solvx                                                                                             | 17.47 MB             | 17.47 MB            
/usr/lib64/libcuda.so.535.146.02                                                                                                    | 17.37 MB             | 28.02 MB            
/root/.vscode-server/data/CachedExtensionVSIXs/ms-python.vscode-pylance-2024.3.1                                                    | 17.31 MB             | 17.31 MB            
/var/lib/plocate/plocate.db                                                                                                         | 14.47 MB             | 14.47 MB            
./cachetop -n 20  5.43s user 6.26s system 98% cpu 11.814 total
```

In contrast to running hcache on the machine at the same time, you can see that hcache is all binaries/dynamic libraries/mappings of the process itself, whereas cachetop actually has access to the entire cache.

<img width="1248" alt="image" src="https://github.com/XinShuichen/cachetop/assets/26585883/f7597e9f-60b0-4188-b6b3-8afee84e0ff7">

<img width="851" alt="image" src="https://github.com/XinShuichen/cachetop/assets/26585883/d26b9768-4fe3-44be-831e-353167c9e889">

