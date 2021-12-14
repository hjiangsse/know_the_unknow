# ssh password-less:
``` bash
#!/bin/sh

help_info() {
    echo "This tool will help you automate the ssh password less process"
    echo "Usage:"
    echo "  ./ssh_hero.sh -u username -m hostname"
    echo "  username: the user your want to ssh to"
    echo "  hostname: the remote mechine you want to ssh to"
}

while getopts hu:m: flag
do
    case "${flag}" in
        u) username=${OPTARG};;
        m) hostname=${OPTARG};;
        h) help_info;;
    esac
done

#gen rsa keys
if [ ! -f ~/.ssh/id_rsa ]; then
    ssh-keygen -t rsa
fi

#create .ssh directory on remote host
ssh ${username}@${hostname} mkdir -p .ssh

#send pub key to remote know hosts
cat ~/.ssh/id_rsa.pub | ssh ${username}@${hostname} 'cat >> .ssh/authorized_keys'

#test if you can login to remote host password less
ssh ${username}@${hostname}
```

# install cmake:
wget https://github.com/Kitware/CMake/releases/download/v3.22.0-rc2/cmake-3.22.0-rc2-linux-x86_64.sh

# iperf (network test tool)
## running as a server
``` bash
iperf -s
```

## running as a client
``` bash
iperf -c 192.168.222.220
``` bash

# install new version gcc and g++:
https://linuxize.com/post/how-to-install-gcc-compiler-on-ubuntu-18-04/

# mmap i/o in the python way:
https://realpython.com/python-mmap/

# compile multi-thread clickhouse c++ client 
``` bash
g++ --std=c++17 multi_thread.cpp -o multi_thread -lclickhouse-cpp-lib -pthread
```

# config git to use sock5 as proxy
set proxy
``` bash
git config --global http.proxy socks5://127.0.0.1:1080
git config --global https.proxy socks5://127.0.0.1:1080
```

to disable the proxy
``` bash
git config --global --unset http.proxy
```
# ssh password-less:
``` bash
#!/bin/sh

help_info() {
    echo "This tool will help you automate the ssh password less process"
    echo "Usage:"
    echo "  ./ssh_hero.sh -u username -m hostname"
    echo "  username: the user your want to ssh to"
    echo "  hostname: the remote mechine you want to ssh to"
}

while getopts hu:m: flag
do
    case "${flag}" in
        u) username=${OPTARG};;
        m) hostname=${OPTARG};;
        h) help_info;;
    esac
done

#gen rsa keys
if [ ! -f ~/.ssh/id_rsa ]; then
    ssh-keygen -t rsa
fi

#create .ssh directory on remote host
ssh ${username}@${hostname} mkdir -p .ssh

#send pub key to remote know hosts
cat ~/.ssh/id_rsa.pub | ssh ${username}@${hostname} 'cat >> .ssh/authorized_keys'

#test if you can login to remote host password less
ssh ${username}@${hostname}
```

# install cmake:
wget https://github.com/Kitware/CMake/releases/download/v3.22.0-rc2/cmake-3.22.0-rc2-linux-x86_64.sh

# iperf (network test tool)
## running as a server
``` bash
iperf -s
```

## running as a client
``` bash
iperf -c 192.168.222.220
``` bash

# install new version gcc and g++:
https://linuxize.com/post/how-to-install-gcc-compiler-on-ubuntu-18-04/

# mmap i/o in the python way:
https://realpython.com/python-mmap/

# compile multi-thread clickhouse c++ client 
``` bash
g++ --std=c++17 multi_thread.cpp -o multi_thread -lclickhouse-cpp-lib -pthread
```
# 
# Linux Kernel
## a wiki introduction:
[linux_kernel_wiki](https://en.wikipedia.org/wiki/Linux_kernel)

# docker
## start two terminal login to the same container
docker run -it ubuntu bash
docker exec -it {docker id} bash
