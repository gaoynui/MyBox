包下载：

wget http://gosspublic.alicdn.com/ossfs/ossfs_1.80.6_centos7.0_x86_64.rpm
yum localinstall  ossfs_1.80.6_centos7.0_x86_64.rpm -y

加入key:

echo  "syncdatacenter:LTAI4G6V2kDRQ8Gqre4HEAGx:3z6WgbVT8J34TOj3Rjhoxe16vTy3zJ" >> /etc/passwd-ossfs
chmod 640 /etc/passwd-ossfs

挂载：

mkdir /data/ossfs
ossfs  bigdata-aibackup /data/ossfs -ourl=oss-cn-shenzhen-internal.aliyuncs.com

