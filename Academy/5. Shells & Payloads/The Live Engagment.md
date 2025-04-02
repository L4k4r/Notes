----

#### WebShell on host1

host1 has an open port 8080. When connecting to the that port via the browser I saw a TomCat server. On the Dekstop of the foothold host was a text file 

*to manage the blog:*
- *admin / admin123!@#  ( keep it simple for the new admins )*

*to manage Tomcat on apache*
- *tomcat / Tomcatadm*

I browsed to the `/manager` directory of TomCat and entered the credentails. I could upload `.war` files. I checked if laudanum had webshells for .war files and he did at `/usr/share/laudanum/jsp/cmd.war`. To get this shell to the foothold host, I setup a python http.server and used `wget` on the foothold host to transfer the files.

After I uploaded the .war file, I browsed to `/cmd/cmd.jsp` to access the shell. To get the hostname I used `systeminfo` to get the files in the `C:\Shares\` directory, I had to use `cmd.exe /c dir c:\Shares\`. 