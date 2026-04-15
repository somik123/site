---
id: 33
title: Basics
type: post
date: 2019-10-08 17:10:23
lastmod: 2023-04-01 23:42:22
categories:
- Bash
---


These is a collection of most basic linux commands. Some require installations, others come pre-installed with the system.



### Show current location:

```
pwd
```



### Show folder size on disk:

```
du -hs /home/somik/dir/
```



### Mirror full websites using wget:

```
wget -e robots=off -m -k https://www.domain.com/
```



### Download everything 2 folders deep on website using wget:

```
wget -e robots=off -m -E -nH -np --cut-dirs=2 http://www.domain.com/folder1/folder2/
```



### Delete a folder:

```
rm -rf /home/somik/dir/
```



### Change ownership of folder:

```
chown -hR www-data:www-data /var/www/html/
```



### RAR a file (must have RAR installed on server):

```
rar a -m0 file.rar "folder1" "folder2" "file3"
```



### UnRAR a file (must have RAR installed on server): 

```
unrar x "file.rar"
```




### tar a file on server:

```
tar -vcf archive.tar 'folder 1' 'folder 2' 'file 3'
```



### tar.gz a file on server:

```
tar -vzcf archive.tar.gz 'folder 1' 'folder 2' 'file 3'
```



### Extract tar file:

```
tar -xvf archive.tar
```



### Extract tar.gz file:

```
tar -zxvf archive.tar.gz
```



### Create a symbolic link:

```
ln -s /home/somik/folder1 /var/www/folder2
```



### View file properties:

```
ls -l
```



### Check server usage by user:

```
ps aux --sort=uid
```


