---
title: My stackoverflow
categories: Debug
date: 2022-09-12
---

很杂,在搜索栏搜搜
<!--more-->

### 重启网络
```bash
sudo service network-manager restart
```

### vim 
**Cut and paste**

    1. Position the cursor where you want to begin cutting.
    2. Press v to select characters, or uppercase V to select whole lines, or Ctrl-v to select rectangular blocks (use Ctrl-q if Ctrl-v is mapped to paste).
    3. Move the cursor to the end of what you want to cut.
    4. Press d to cut (or y to copy).
    5. Move to where you would like to paste.
    6. Press P to paste before the cursor, or p to paste after.

**Copy and paste is performed with the same steps except for step 4 where you would press y instead of d**

    * d stands for delete in Vim, which in other editors is usually called cut
    * y stands for yank in Vim, which in other editors is usually called copy


### VScode

**Remove software**
If you installed via Snap:
```bash
    $sudo snap remove vscode
```
If you installed via apt:
```bash
    $sudo apt-get purge code
```
If you installed via Ubuntu Software, open Ubuntu Software, look for the app in the installed category, and click on remove.

**Remove settings**
```bash
$cd ~ && rm -rf .vscode && rm -rf .config/Code

```

### Tex
**Making CV with resumeSubheading mode**
[How to add another line](https://tex.stackexchange.com/questions/407982/adding-an-another-line)

### C++
#### Vector
Copy vector to array, do not use 
```c
#include <vector>
double * a;
std::vector<double> v;
a = v.data();
```
Thus, the pointer `a` = &v[0], and you lose the address allocated to `a` before. Why did I just find this problem?? Because I was using `cudaMallocManaged`
```c
double * a;
cudaMallocManaged(&a, m_n * sizeof(double));
std::copy(v.begin(), v.end(), a);
```
After allocating the memory to `a` using Unified Memory system, if I use `a = v.data()`, it is lost!

#### CUDA
When I declared an array in cuda kernel like this:
```c
{
    ...
    int thread_id = blockDim.x * blockIdx.x + threadIdx.x;
    double * shared = new double[nonzero];
    if (thread_id < nonzero){
      shared[thread_id] = x[col[thread_id]]*value[thread_id]; 
      atomicAdd(&Ap[row[thread_id]], shared[thread_id]);
    ...
}
```
After several calls to the kernel, it will crash. It is indeed awful code, but I don't know it is fatally wrong. I kept `shared` because I wanted to try for shared memory later and remove atomic operation.

(I DON'T KNOW WHY I AM ALWAYS WASTING TIME ON THE STUPID, UNNECESSARY MISTAKES!)


#### const pointer and pointer to const

[Clockwise/Sprial Rule](http://c-faq.com/decl/spiral.anderson.html)

<https://stackoverflow.com/questions/1143262/what-is-the-difference-between-const-int-const-int-const-and-int-const>


### apt-get报错
1. 
```
E: Encountered a section with no Package: header
E: Problem with MergeList /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_natty_main_binary-i386_Packages
E: The package lists or status file could not be parsed or opened.
```
[How do I fix a "Problem with MergeList" or "status file could not be parsed" error when trying to do an update?](https://askubuntu.com/questions/30072/how-do-i-fix-a-problem-with-mergelist-or-status-file-could-not-be-parsed-err)

2. 
[How can I fix apt error "W: Target Packages ... is configured multiple times"?](https://askubuntu.com/questions/760896/how-can-i-fix-apt-error-w-target-packages-is-configured-multiple-times)

3.
[Ubuntu apt-get彻底卸载软件包](https://www.jianshu.com/p/4a409053575a)

### ssh连接
1. hostname太长了怎么办，比如icculster???.iccluster.xxxx.xxx

在`~/.ssh/config`里面添加
```
Host iccluster*
Hostname %h.iccluster.xxxx.xxx
User [username]
```
然后ssh　icclusterxxx就可以了

用`ssh-keygen`生成一对public/private key(e.g. axiwa_id.pub/axiwa_id)，把`.pub`放到remote machine的`.ssh/autorized_keys里，`~/ssh/config`里面添加`IdentityFile ~/.ssh/axiwa_id`

### Ubuntu 20 无法使用虚拟机:
最完整有用的办法！<http://askonline.tech/ask/9339.html>

### 虚拟机无法使用shared文件夹
首先自己的host machine上装好.iso<https://askubuntu.com/questions/22743/how-do-i-install-guest-additions-in-a-virtualbox-vm>

然后给虚拟机挂好，虚拟机挂不好说明虚拟机的虚拟机需要更新<https://superuser.com/questions/760327/vbox-guest-additions-iso-cant-be-mounted-because-of-verr-pdm-media-locked>

网友都太厉害了


### NVIDIA DRIVER AND CUDA
<https://www.cnblogs.com/pprp/p/9430836.html> 驱动安装

<https://ubuntu.com/blog/how-to-sign-things-for-secure-boot>安全启动模式下怎么添加key


### 怎么用命令行进行格式化
先卸载，再格式化

卸载:
```bash
sudo umount /media/disk
```

格式化:

知道自己的U盘叫什么
```bash
sudo fdisk -l
```
device: /dev/sdb

格式化
```bash
sudo mkfs.vfat /dev/sdb
```


### Change g++/gcc version 

<https://askubuntu.com/questions/26498/how-to-choose-the-default-gcc-and-g-version>

### How to find which CPU core a process is running on

<https://www.xmodulo.com/cpu-core-process-is-running.html>



### Perl&bash
<https://linuxconfig.org/how-to-propagate-a-signal-to-child-processes-from-a-bash-script>

### Ubuntu的设置栏不见了
<https://serverok.in/ubuntu-20-04-settings-wont-open>

