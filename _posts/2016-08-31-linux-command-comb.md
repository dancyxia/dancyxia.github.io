---
layout: post
title: Linux Development - curl usage
category: linux-dev
tags: [linuxdev]
---

<style>
table{
width: 100%;
table-layout:fixed;
border-collapse: collapse;
border-spacing: 0;
border:2px solid #000000;
}

th{
border:2px solid #000000;
}

td{
word-wrap:break-word;
border:1px solid #000000;
}

td:nth-child(1), th:nth-child(1){
    width: 20%;
}
td:nth-child(2){
    width: 30%;
}
td:nth-child(3){
    width: 50%;
}
</style>

# Linux command

### curl usage


[//]:  <table>
[//]: <tr>
[//]: <td></td>
[//]: </tr>
[//]: </table>

|-------------------+--------|
| command | usage  |
| -------- | ----- |
|  find the longest filename |ls index* \|awk '{if (length($0) > max) max=length($0) } END {print max}'|


print message by ignoring the first three columns 
grep ERROR /log/csc.log |awk '{$1=$2=$3=""}1'

replace all with 
find . -type f -exec sed -i 's/foo/bar/g' {} +
| Get a file from an SSH server using SCP using a private key(password-protected) to authenticate |
|          | -------- | ---------- |
|| curl -u username: --key ~/.ssh/id_rsa scp://example.com/~/file.txt | Get a file from an SSH server using SCP using a private key(not password-protected) to authenticate |
|--------- |--------- | ---------- |
| GET      | curl -u name:passwd ftp://machine.domain:port/full/path/to/file | get file from ftp server |
|          |------------+-------------|
|          | curl -u user:passwd -x my-proxy:888 http://www.get.this/ | get file from http server that requires user/pass through proxy |
|          |------------+-------------|
  |          | curl -U user:passwd -x my-proxy:888 http://www.get.this/ | get file from http server through proxy that requires user/pass |
  |          |------------+-------------|
  |          | curl --noproxy localhost,get.this -x my-proxy:888 http://www.get.this/ | A comma-separated list of hosts and domains which do not use the proxy|
  |          +------------+-------------|
  |          |  curl -o thatpage.html http://www.netscape.com/ | download to a file |
  |          +------------+-------------|
  |          | curl -O www.haxx.se/index.html -O curl.haxx.se/download.html | Fetch two files and store them with their remote names |
  |--------- +------------+-------------|
  | UPLOADING | curl -T - ftp://ftp.upload.com/myfile |  Upload all data on stdin to a specified server |
  |--------- +------------+-------------|
  | FTP / FTPS / SFTP / SCP | curl -T uploadfile -u user:passwd ftp://ftp.upload.com/myfile | Upload data from a specified file, login with user and password |
  |--------- +------------+-------------|
  | POST | curl -d "name=Rafael%20Sagula&phone=3320780" http://www.where.com/guest.cgi | Post a simple "name" and "phone" guestbook. |
  |------+ -------------------------------+ -------------|
  |      |   curl -F "coolfiles=@fil1.gif;type=image/gif,fil2.txt,fil3.html" http://www.post.com/postit.cgi | Upload a file |
  |         + -------------------------------+ -------------|
  |      | curl -F "file=@cooltext.txt" -F "yourname=Daniel" -F "filedescription=Cool text file with cool text inside" http://www.post.com/postit.cgi | Emulate a fill-in form |
  |------+ -------------------------------+ -------------|
  |  | curl -F "pictures=@dog.gif,cat.gif" | Send multiple files in a single "field" with a single field name |
  |------+ -------------------------------+ -------------|
  |  | curl -F "docpicture=@dog.gif" -F "catpicture=@cat.gif" | Send two fields with two field names |
  |------+ -------------------------------+ -------------|
  |Cookie  |curl --dump-header headers www.example.com | dump headers |
  |------+ -------------------------------+ -------------|
  |  | curl -b headers www.example.com | use the cookies from the 'headers' file|
  |  | curl -c cookies.txt www.example.com | save the incoming cookies |
  |  | curl -L -b empty.txt www.example.com | by specifying -b you enable the "cookie awareness" and with -L you can make curl follow a location. if a site sends cookies and a location, you can use a non-existing file to trigger the cookie awareness |
  |-----+------+-----|
  | EXTRA HEADERS| curl -H "X-you-and-me: yes" www.love.com | send the header "X-you-and-me: yes" to the server when getting a page|
  | SFTP and SCP | curl -u $USER sftp://home.example.com/~/.bashrc | Get .bashrc from your home director |
  | FTP |curl ftp://user:passwd@my.site.com/README | get the file 'README' from your home directory at your ftp site|
  |   |curl ftp://user:passwd@my.site.com//README | want the README file from the root directory of the ftp site|
  |   | curl -E /path/to/cert.pem:password https://secure.site.com/ | https with given certificate | 
  |=



#### Reference

**Reference 1:** [manual site](https://curl.haxx.se/docs/manual.htm)
