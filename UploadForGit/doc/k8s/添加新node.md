## 在master上生成新的token
kubeadm token create --print-join-command
## 在master上生成用于新master加入的证书
kubeadm init phase upload-certs --upload-certs
## 输入token添加新node
kubeadm join ...
