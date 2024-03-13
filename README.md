# cachetop

List the files in the cache that use the most memory. Low memory usage, automatic exclusion of virtual filesystems such as /proc, and no unowned files are missed

## Usage

WIP, Please modify scan folder in main func. now is "/".
Then, run `./cachetop`

```
$ time ./cachetop
FILE NAME                                                                                                                           		CACHE SIZE           FILE SIZE
/usr/lib/sysimage/rpm/rpmdb.sqlite                                                                                                  		170.86 MB            174.05 MB
/var/cache/dnf/fedora-filenames.solvx                                                                                               		53.64 MB             53.66 MB
/var/cache/dnf/fedora-376ef8e983c65ce0/repodata/5f86dfe1903316d6b25d494616cf18352204fe70c529b4c97859df6faecd493f-filelists.xml.zck  		49.40 MB             49.40 MB
/usr/lib64/libclang-cpp.so.16                                                                                                       		45.96 MB             66.50 MB
/usr/lib64/libLLVM-16.so                                                                                                            		42.55 MB             117.79 MB
/var/cache/dnf/updates-filenames.solvx                                                                                              		34.73 MB             34.73 MB
/var/cache/dnf/fedora-376ef8e983c65ce0/repodata/dfa2aec8d2e83459677d698542f1a190d92815888668d027b21dfd46fb86ce01-primary.xml.zck    		31.68 MB             31.68 MB
/var/log/journal/4483974d999a4003bae5281e394062bb/system.journal                                                                    		31.02 MB             32.00 MB
/var/cache/dnf/updates-b7ba662710b98f1a/repodata/beb7205a057a206bfae959a3f15f456dd9e3f12da78ce8a9964958fdf1a8ad6e-filelists.xml.zck 		30.06 MB             30.06 MB
/root/anaconda3/libexec/gcc/x86_64-conda-linux-gnu/11.2.0/cc1                                                                       		29.62 MB             29.62 MB
/root/anaconda3/pkgs/gcc_impl_linux-64-11.2.0-h1234567_1/libexec/gcc/x86_64-conda-linux-gnu/11.2.0/cc1                              		29.62 MB             29.62 MB
/usr/lib/locale/locale-archive                                                                                                      		25.98 MB             213.97 MB
/usr/lib/locale/locale-archive.real                                                                                                 		25.98 MB             213.97 MB
/var/cache/dnf/fedora.solv                                                                                                          		25.49 MB             25.99 MB
/var/lib/docker/overlay2/469328597deff3ff11e139b1b279dd15f802ec66c27984be99877e833bec407f/diff/coze-discord-proxy                   		18.50 MB             28.84 MB
/root/jellyfin/config/data/library.db                                                                                               		17.88 MB             17.88 MB
/var/cache/dnf/updates-updateinfo.solvx                                                                                             		17.45 MB             17.45 MB
/root/.vscode-server/data/CachedExtensionVSIXs/ms-python.vscode-pylance-2024.3.1                                                    		17.31 MB             17.31 MB
/var/lib/plocate/plocate.db                                                                                                         		14.47 MB             14.47 MB
/var/lib/docker/overlay2/5c97ebae1f6c4e8942b239302bc0014ef8703a511dc595fbe8297626989b4eec/diff/kavita/System.Private.CoreLib.dll    		12.28 MB             12.28 MB
./cachetop  5.49s user 6.13s system 98% cpu 11.751 total
```
