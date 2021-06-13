# soal-shift-sisop-modul-4-I03-2021

### **Number 1**
a.	If a directory made with `AtoZ_` with its front, then the file will be encoded
b.	If its rename like in in a point, then it will be encoded
c.	If the encrypted directory renamed, the directory contains will decoded
d.	mkdir and rename directory will be noted to log
e.	encoding method will be applied in the contains of the directory, and will be done recursively

The problem ask us to do encrypt and decrypt file name and folder using Atbash cipher. To get the file and folder we use loop. There is 3 function that we use in here. `slash_id` (to return index slash), `ext_id`(return the index file extension), and `split_ext_id` (to return the index file extension in the splitted file). 
```c
int split_ext_id(char *path)
{
int ada = 0;
	for(int i=strlen(path)-1; i>=0; i--){
		if (path[i] == '.'){
			if(ada == 1) return i;
			else ada = 1;
		}
	}
	return strlen(path);
}

int ext_id(char *path)
{
for(int i=strlen(path)-1; i>=0; i--){
		if (path[i] == '.') return i;
	}
	return strlen(path);
}

int slash_id(char *path, int mentok)
{
for(int i=0; i<strlen(path); i++){
		if (path[i] == '/') return i + 1;
	}
	return mentok;
}
```
To encrypt, we use Atbash function
```c
void encryptAtbash(char *path)
{
if (!strcmp(path, ".") || !strcmp(path, "..")) return;
	
	printf("encrypt path Atbash: %s\n", path);
	
	int endid = split_ext_id(path);
	if(endid == strlen(path)) endid = ext_id(path);
	int startid = slash_id(path, 0);
	
for (int i = startid; i < endid; i++){
		if (path[i] != '/' && isalpha(path[i])){
			char tmp = path[i];
			if(isupper(path[i])) tmp -= 'A';
			else tmp -= 'a';
			tmp = 25 - tmp; //Atbash cipher
			if(isupper(path[i])) tmp += 'A';
			else tmp += 'a';
			path[i] = tmp;
		}}}
```
To decrypting is similar to the function above
```c
void decryptAtbash(char *path)
{
if (!strcmp(path, ".") || !strcmp(path, "..")) return;
	
	printf("decrypt path Atbash: %s\n", path);
	
	int endid = split_ext_id(path);
	if(endid == strlen(path)) endid = ext_id(path);
	int startid = slash_id(path, endid);
	
for (int i = startid; i < endid; i++){
		if (path[i] != '/' && isalpha(path[i])){
			char tmp = path[i];
			if(isupper(path[i])) tmp -= 'A';
			else tmp -= 'a';
			tmp = 25 - tmp; //Atbash cipher
			if(isupper(path[i])) tmp += 'A';
			else tmp += 'a';
			path[i] = tmp;
		}}}
```
Decryption will be executed in each utility function such as `mkdir`.
Decrypt and encrypt will be called in `readdir` function. While calling the function, it will check `AtoZ_` string in the path of each utility function by using `strstr`. Logging will also noted, but it will explained later in number 4. ( the logging of `mkdir` and `rename` is still in problem and undone)

Difficulties
-	Encrypt and decrypt could be executed as 1 function, but we it was already done
-	Logging the mkdir and rename is still in problem

### **Number 2**
This problem requires us to make more encryption.
a.	If the directory created with a name format `RX_..`, the directory will be encoded with its containments, additionally with ROT13 algorithm
b.	If its renamed with `RX_...`, it will be encoded. With addition using Vignere Cipher with key `SISOP`. It is case sensitive
c.	If we rename encrypted directory, it will decoded
d.	All encoding will be noted to logged
e.	Files in directory will split into 1024 bytes size files. When accessed thru filesystem it only appeared as `Suatu_File.txt`

In `xmp_rename`, it will check if the directory already been rename by adding `RX_` or delete the `RX_` using `strstr`.

```c
static int xmp_rename(const char *from, const char *to)
{
	int res;
	char frompath[1000], topath[1000];
	
	char *a = strstr(to, atoz);
	if (a != NULL) decryptAtbash(a);
	
	char *b = strstr(from, rx);
	if (b != NULL){
		decryptRot13(b);
		decryptAtbash(b);
	}
	
	char *c = strstr(to, rx);
	if (c != NULL){
		decryptRot13(c);
		decryptAtbash(c);
	}

	sprintf(frompath, "%s%s", dirPath, from);
	sprintf(topath, "%s%s", dirPath, to);

	res = rename(frompath, topath);
	if (res == -1) return -errno;

	tulisLog2("RENAME", frompath, topath);
	
	if (c != NULL){
		enkripsi2(topath);
		tulisLog2("ENCRYPT2", from, to);
	}

	if (b != NULL && c == NULL){
		dekripsi2(topath);
		tulisLog2("DECRYPT2", from, to);
	}

```

If `RX_` is included in the path, it will be renamed by adding `RX_ `. File split will be accompany the progress (although the file splitting is still in problem).
If `RX_` detected in original path and undetected in `RX_` in destination path, the directory already been renamed by delete the `RX_`, then it will be continued to integrate the files in `deskrpsi2` (the integration is still undone).


To encrypt and decrypt, we use ROT13 cipher to make another function
```c
void encryptRot13(char *path)
{
	if (!strcmp(path, ".") || !strcmp(path, "..")) return;
	
	printf("encrypt path ROT13: %s\n", path);
	
	int endid = split_ext_id(path);
	int startid = slash_id(path, 0);
	
	for (int i = startid; i < endid; i++){
		if (path[i] != '/' && isalpha(path[i])){
			char tmp = path[i];
			if(isupper(path[i])) tmp -= 'A';
			else tmp -= 'a';
			tmp = (tmp + 13) % 26; //ROT13 cipher
			if(isupper(path[i])) tmp += 'A';
			else tmp += 'a';
			path[i] = tmp;
		}
	}
}

void decryptRot13(char *path)
{
	if (!strcmp(path, ".") || !strcmp(path, "..")) return;
	
	printf("decrypt path ROT13: %s\n", path);
	
	int endid = split_ext_id(path);
	int startid = slash_id(path, endid);
	
	for (int i = startid; i < endid; i++){
		if (path[i] != '/' && isalpha(path[i])){
			char tmp = path[i];
			if(isupper(path[i])) tmp -= 'A';
			else tmp -= 'a';
			tmp = (tmp + 13) % 26; //ROT13 cipher
			if(isupper(path[i])) tmp += 'A';
			else tmp += 'a';
			path[i] = tmp;
		}
	}
}
```
The solution is quite similar to number 1, but with additional file split that we still cannot be solve by us. The logging is also still in problem because its still un-noted the `mkdir` and `rename`

Difficulties
-	Logging the `mkdir` and `rename` is still in problem
-	We have problems in splitting the files


### **Number 3**

Still unsolvable

### **Number 4**

This problem require us to logging all progress from the functions above.

a.	Log system will be made in home directory. It will contain all system call in the file system
b.	There is two type level, `INFO` and `WARNING`
c.	WARNING cited syscall `rmdir` and `unlink`
d.	The rest will cited as`INFO`
e.	Formatted as in the problem

To log it, we use`tulislog`and`tulislog2`functions. First we need to find the time info
```c
void tulisLog(char *nama, char *fpath)
{
	time_t rawtime;
	struct tm *timeinfo;
	time(&rawtime);
	timeinfo = localtime(&rawtime);
```
we will initialize an array to store a syscall demand that has already been executed in filesystem and note it in`SinSeiFS.log` we need to open the file in home with mode append so that it could be written. then we check the syscall. If its`RMDIR`or`UNLINK` it will be noted as`WARNING`.
The other syscall will noted as `INFO`.
To write and close the file we use`fputs`and`fclose`

```
char haha[1000];
	
	FILE *file;
	file = fopen("/home/aldo/SinSeiFS.log", "a");
……
if (strcmp(nama, "RMDIR") == 0 || strcmp(nama, "UNLINK") == 0)
		sprintf(haha, "WARNING::%.2d%.2d%d-%.2d:%.2d:%.2d::%s::%s\n", timeinfo->tm_mday, timeinfo->tm_mon + 1, timeinfo->tm_year + 1900, timeinfo->tm_hour, timeinfo->tm_min, timeinfo->tm_sec, nama, fpath);
	else
		sprintf(haha, "INFO::%.2d%.2d%d-%.2d:%.2d:%.2d::%s::%s\n", timeinfo->tm_mday, timeinfo->tm_mon + 1, timeinfo->tm_year + 1900, timeinfo->tm_hour, timeinfo->tm_min, timeinfo->tm_sec, nama, fpath);
……

fputs(haha, file);
	fclose(file);
	return;
}
```
In`tulislog2` is quite similar to`tulislog` But there are some parameters difference.
```c
void tulisLog2(char *nama, const char *from, const char *to)
{
	time_t rawtime;
	struct tm *timeinfo;
	time(&rawtime);
	timeinfo = localtime(&rawtime);

	char haha[1000];

	FILE *file;
	file = fopen("/home/aldo/SinSeiFS.log", "a");

	if (strcmp(nama, "RMDIR") == 0 || strcmp(nama, "UNLINK") == 0)
		sprintf(haha, "WARNING::%.2d%.2d%d-%.2d:%.2d:%.2d::%s::%s::%s\n", timeinfo->tm_mday, timeinfo->tm_mon + 1, timeinfo->tm_year + 1900, timeinfo->tm_hour, timeinfo->tm_min, timeinfo->tm_sec, nama, from, to);
	else
		sprintf(haha, "INFO::%.2d%.2d%d-%.2d:%.2d:%.2d::%s::%s::%s\n", timeinfo->tm_mday, timeinfo->tm_mon + 1, timeinfo->tm_year + 1900, timeinfo->tm_hour, timeinfo->tm_min, timeinfo->tm_sec, nama, from, to);

	fputs(haha, file);
	fclose(file);
	return;
}
```
Difficulties:
-mkdir and rename log is still not written
