# K8S 환경 설정

1. 호스트 네임 등록  
    `echo "127.0.0.1 $HOSTNAME" >> /etc/hosts`

2. 패키지 설치
  
    iptables  
    `sudo yum install -y iptables`  
    패킷 필터링과 NAT 도구로, 네트워크 플러그인의 트래픽 필터링을 위해 필요로 함
    iproute-tc  
    `yum install iproute-tc -y`  
    트래픽 제어 관리 도구로, Calico와 같은 네트워크 플러그인의 패킷 흐름제어를 위해서 필요함

    ipv4 포워드 설정 변경

    ```bash
    echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
    sudo sysctl -p
    ```  

    파드간의 트래픽 라우트를 위해서 필요함

3. Cotaniner Management Tool 설치 (containerd)  
    `yum install containerd -y`

    3-1. Default Config 파일 생성  
    `containerd config default > /etc/containerd/config.toml`

    3-2. Cgroup 관련 설정  
    `sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml`

    3-3. Containerd 실행  

    ```bash
    systemctl enable containerd
    systemctl start containerd
    systemctl status containerd
    ```

4. Bridge 네트워크 세팅  

    ```bash
    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF
    
    sudo modprobe overlay
    sudo modprobe br_netfilter
    ```

    `overlay` 서로 다른 노드의 파드간의 통신을 가능하게 해주는 모드
    `br_netfilter` 브릿지 인터페이스를 통한 패킷 필터링 지원 활성화

    ```bash
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF
    ```

    브릿지 네트워크는 디폴트로 iptables를 우회하도록 설정되어 있지만, 컨테이너의 네트워크 패킷이 호스트 머신의 iptables 설정을 따라 제어되도록 재설정

5. System 설정 리로드  
    `sysctl --system`

6. Swap 비활성화  
    `swapoff -a`

7. SELinux permissive 모드로 전환  

    ```bash
    setenforce 0
    sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
    ```

8. K8S 리소스 레포지토리 등록  

    ```bash
    cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/
    enabled=1
    gpgcheck=1
    gpgkey=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/repodata/repomd.xml.key
    exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
    EOF
    ```

9. Yum install  
    `yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes`

10. kubelet 서비스 활성화  
    `systemctl enable --now kubelet`

11. Local User kubelet 액세스 활성화  

    ```bash
    mkdir -p $HOME/.kube
    cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    chown $(id -u):$(id -g) $HOME/.kube/config
    ```

12. calico 네트워크 플러그인 설치

    [calico docs](https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart#install-calico)
