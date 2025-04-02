----
SMB is commonly used in Windows networks, and we will often find share folders in a Windows network. We can interact with SMB using the GUI, CLI or tools.


- **Windows GUI**
	Press `[WINKEY] + R` and type the file share location:
		`\\192.168.220.129\Finance\`
	If we don't have access, we will receive an authentication request.

- **Windows CMD**
	The command `dir` displays a list of directory's and subdirectories:
		`dir \\192.168.220.129\Finance\`
	
	The command `net use` connects to a computer from a shared resource. We have to map its content to a drive letter `n`
		`net use n: \\192.168.220.129\Finance\`
	We can also provide credentials to authenticate
		`net use n: \\192.168.220.129\Finance\ /user:plaintext Password123`

- **Get info about directory / drive**
	To see how many files the shared folder and its subdirectory contain:
		`dir n: /a-d /s /b | find /c ":\"`
	- **dir** - Application
	- **n:** - Directory or drive to search
	- **/a-d** - **/a** is the attribute and **-d** means not directories 
	- **/s** - Display files in a specified directory and all subdirectories
	- **/b** - Use bare format (no heading information or summary)
	The command `| find /c ":\\"` process the output of the `dir` command to count how many files exist in the directory and subdirectories.
- **Find file names**
	With `dir` we can search for specific names in files
		`dir n:\*cred* /s /b`
		`dir n:\*secret* /s /b`
- **Find specific word within a file**
		`findstr /s /i cred n:\*.*`


- **Windows PowerShell**
	Get contents of shared folder
		`Get-ChildItem \\192.168.220.129\Finance`
	Instead of `net use` we can use `New-PSDrive`
		`New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance -PSProvider "FileSystem"`
	To provide a username and password, we need to create a PSCredential object
		`$username = 'plaintext'`
		`$password = 'Password123'`
		`$secpassword = ConvertTo-SecureString $password -AsPlainText -Force`
		`$cred = New-Object System.Management.Automation.PSCredential $username, $password`
		`New-PSDrive -Name "N" -Root \\192.168.220.129\Finance -PSProvider "FileSystem" -Credential $cred`
	
- **Count files**
	Use the command `Get-ChildItem` or `GCI` instead of `dir`
		`(Get-ChildItem -File -Recurse | Measure-Object).Count`
	We can use the `-Include` to find specific items from the directory
		`Get-ChildItem -Recurse -Path N:\ -Include *cred* -File`
	To find text patterns in files
		`Get-ChildItem -Recurse -Path N:\ | Select-String "cred" -List`


- **Linux Mount**
	Mount SMB shares to interact with directories and files locally
		`sudo mkdir /mnt/Finance`
		`sudo mount -t cifs -o username=plaintext,password=Password123,domain=. //192.168.220.129/Finance /mnt/Finance`
	Find filenames that contains specific strings
		`find /mnt/Finance/ -name *cred*`
	Find files that contain the specific string
		`grep -rn /mnt/Finance/ -ie cred`