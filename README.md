# sr_iov_prac

test 환경
--------
  
        os : centos7.8
        mlnx-ofed : 4.9(LTS)
        vm : virt-manager
        
        vm 과 host 조건 동일
        host는 gss-manager(192.168.21.191) 노드
        

bios setting
------------
        sr-iov 활성화
        1. bios 설정 접속
        2. System Settings
        3. Network
        4. network device list에서 infiniband device 선택
        5. Mellanox Network Adapter 선택
        6. Virtualization Mode <SR-IOV> 선택
        7. 디바이스 별로 동일
        
grub파일 수정
-----------
        1. /etc/default/grub 파일에서 GRUB_CMDLINE_LINUX에 'intel_iommu=on iommu=pt' 추가
        2. grub2-mkconfig -o /boot/grub2/grub.cfg 실행
        

mlnx_ofed 설치
-------------
        1. MLNX_OFED_LINUX-4.9-0.1.7.0-rhel7.8-x86_64.tgz 다운로드
        2. tar -xvf MLNX_OFED_LINUX-4.9-0.1.7.0-rhel7.8-x86_64.tgz
        3. cd MLNX_OFED_LINUX-4.9-0.1.7.0-rhel7.8-x86_64
        4. ./mlnxofedinstall --add-kernel-support
        5. /etc/init.d/openibd restart
        6. /etc/init.d/opensmd restart
        
SR-IOV configuration
--------------------
        1. mst start
        2. mst status 로 디바이스 확인
        3. mlxconfig -d /dev/mst/mt4099_pciconf0 set SRIOV_EN=1 NUM_OF_VFS=4 /// SRIOV_EN는 sr-iov활성화 NUM_OF_VFS는 만들 virtual function의 개수
        4. reboot
        5. mst start 
        6. mlxconfig -d /dev/mst/mt4099_pciconf0 q 로 위에서 설정한 parameter 설정 됐는지 확인
        
enabling SR-IOV on MLNX_OFED driver
-----------------------------------
        1. vi /etc/modprobe.d/mlx4_core.conf 
        2. options mlx4_core num_vfs=4 port_type_array=1,1 probe_vf=0   /// VF는 virtual function의 약자
        
        # num_vfs - is the number of VF required for this server, in this example 4 VFs.
        # port_type_array - is the port type of the interface, 1 is for infiniBand, 2 for Ethernet
        # probe_vf - is the number of VF to be probed in the hypervisor
        
        3. /etc/init.d/openibd restart
        4. lspci | grep Mellanox로 VF확인 가능
        
        
 lspci의 예시
        
        # 03:00.0 Network controller: Mellanox Technologies MT27500 Family [ConnectX-3]

        # 03:00.1 Network controller: Mellanox Technologies MT27500 Family [ConnectX-3 Virtual Function]

        # 03:00.2 Network controller: Mellanox Technologies MT27500 Family [ConnectX-3 Virtual Function]

        # 03:00.3 Network controller: Mellanox Technologies MT27500 Family [ConnectX-3 Virtual Function]

        # 03:00.4 Network controller: Mellanox Technologies MT27500 Family [ConnectX-3 Virtual Function]
        
       
VM에서 pci추가
------------
        1. lspci | grep Mellanox에서 볼 수 있는 pci address 토대로 device 추가


vm에서 IB설정
-----------
        1. host와 동일한 mnlx_ofed설치
        2. ib interface에 주소 부여
        3. ibdev2netdev
        # mlx4_0 port 1 ==> ib0 (Up)

        # mlx4_0 port 2 ==> ib1 (Down)
        
        
 주의 사항
 -------
        1. host에서 ib의 state가 initializing 에서 멈춰있는 경우 Open Subnet Manager를 실행 - 
        # opensm -c opensm.conf
        # vi opensm.conf
        subnet_prefix 부분 수정 # 서브넷은 각 IBA Interface 마다 달라야한다
        # opensm -F opensm.conf
