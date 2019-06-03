[TOC]

In linux world, archiving and compression are two separate procedures:
- tar: puts multiple files into a single (tar) file
- gzip: a specific compression method to compress one file (only)

So `tar` command bundles all files (or directories) into a single file and then you specify what method to compress the one single file. With different compression method, the resulting file usually ends with different file extension, like `example.tar.gz` (sometimes `example.tgz`), the `.gz` means this archive is compressed through `gzip` command. So if there is only single file to be compressed, there is no need to use `tar` command, since you could simply compress the file. Refer to this [question](https://askubuntu.com/questions/122141/whats-the-difference-between-tar-gz-and-gz-or-tar-7z-and-7z).

# Using `tar` command
Refer to this [blog](https://support.hostway.com/hc/en-us/articles/360000263544-How-to-compress-and-extract-files-using-tar-command-in-Linux) for more information.

- Compress an entire directory or a single file
  - `tar -czvf name-of-archive.tar.gz /path/to/directory-or-file`
  - `-c`: **c**reate an archive
  - `-z`: compress the archive with g**z**ip
  - `-v`: **v**erbose
  - `-f`: allows you to sepcify the **f**ilename of the archive.
  - You could specify more than one file or directory: `tar -czvf archive.tar.gz foldA/ foldB/ fileA fileB`
- Extract an archive
  - `tar -xzvf name-of-archive.tar.gz`: extract the archive in the current directory
  - Use `-C` to extract archive to specific directory: `tar -xzvf name-of-archive.tar.gz -C /tmp`
  - **It seems that the command itself will try to infer the correct compression method to uncompress the file, maybe from its file extension. So there is no need to specify the compression method. Just `tar -xvf name-of-archive.tar.gz` is fine too.**
- Simply list the content in the archive
  - `tar -tvf name-of-archive.tar.gz`: use the option `-t` to list the contents.
