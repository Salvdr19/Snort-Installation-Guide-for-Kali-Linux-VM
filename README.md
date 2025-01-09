### Snort Installation Guide for Kali Linux (Virtual Machine)

#### **Introduction**
This guide provides a step-by-step walkthrough for installing and configuring Snort 3 on a Kali Linux virtual machine. It covers the installation of prerequisites, disabling network offloads, installing Snort, and integrating PulledPork for automated rule management. This setup is ideal for cybersecurity enthusiasts and professionals looking to build an IDS lab environment.

---

### **1. System Preparation**

#### **Update System and Install Prerequisites**
1. Update and upgrade system packages:
   ```bash
   sudo apt-get update && sudo apt-get upgrade -y
   ```
2. Install the necessary packages:
   ```bash
   sudo apt-get install -y build-essential autotools-dev libdumbnet-dev libluajit-5.1-dev libpcap-dev \
   zlib1g-dev pkg-config libhwloc-dev cmake liblzma-dev openssl libssl-dev cpputest libsqlite3-dev \
   libtool uuid-dev git autoconf bison flex libcmocka-dev libnetfilter-queue-dev libunwind-dev \
   libmnl-dev ethtool libjemalloc-dev
   ```

---

### **2. Install Dependencies**

#### **PCRE (Perl Compatible Regular Expressions)**
1. Navigate to the working directory:
   ```bash
   mkdir -p ~/snort && cd ~/snort
   ```
2. Download and install PCRE:
   ```bash
   wget https://sourceforge.net/projects/pcre/files/pcre/8.45/pcre-8.45.tar.gz
   tar -xzvf pcre-8.45.tar.gz
   cd pcre-8.45
   ./configure
   make
   sudo make install
   ```

#### **Additional Dependencies**
Repeat similar steps for other dependencies:

- **gperftools:**
   ```bash
   cd ~/snort
   wget https://github.com/gperftools/gperftools/releases/download/gperftools-2.9.1/gperftools-2.9.1.tar.gz
   tar xzvf gperftools-2.9.1.tar.gz
   cd gperftools-2.9.1
   ./configure
   make
   sudo make install
   ```

- **Ragel:**
   ```bash
   cd ~/snort
   wget http://www.colm.net/files/ragel/ragel-6.10.tar.gz
   tar -xzvf ragel-6.10.tar.gz
   cd ragel-6.10
   ./configure
   make
   sudo make install
   ```

- **Boost C++ Libraries:**
   ```bash
   cd ~/snort
   wget https://boostorg.jfrog.io/artifactory/main/release/1.77.0/source/boost_1_77_0.tar.gz
   tar -xvzf boost_1_77_0.tar.gz
   ```

- **Hyperscan:**
   ```bash
   cd ~/snort
   wget https://github.com/intel/hyperscan/archive/refs/tags/v5.4.2.tar.gz
   tar -xvzf v5.4.2.tar.gz
   mkdir ~/snort/hyperscan-5.4.2-build
   cd hyperscan-5.4.2-build/
   cmake -DCMAKE_INSTALL_PREFIX=/usr/local -DBOOST_ROOT=~/snort/boost_1_77_0/ ../hyperscan-5.4.2
   make
   sudo make install
   ```

- **Flatbuffers:**
   ```bash
   cd ~/snort
   wget https://github.com/google/flatbuffers/archive/refs/tags/v2.0.0.tar.gz -O flatbuffers-v2.0.0.tar.gz
   tar -xzvf flatbuffers-v2.0.0.tar.gz
   mkdir flatbuffers-build
   cd flatbuffers-build
   cmake ../flatbuffers-2.0.0
   make
   sudo make install
   ```

- **DAQ (Data Acquisition):**
   ```bash
   cd ~/snort
   wget https://github.com/snort3/libdaq/archive/refs/tags/v3.0.13.tar.gz -O libdaq-3.0.13.tar.gz
   tar -xzvf libdaq-3.0.13.tar.gz
   cd libdaq-3.0.13
   ./bootstrap
   ./configure
   make
   sudo make install
   ```

---

### **3. Disable LRO & GRO**

#### **Manually Disable**
1. Identify the network adapter:
   ```bash
   ip a
   ```
2. Turn off LRO and GRO:
   ```bash
   sudo ethtool -K <network_adapter> gro off
   sudo ethtool -K <network_adapter> lro off
   ```

#### **Automate with a Systemd Service**
1. Create a service file:
   ```bash
   sudo nano /etc/systemd/system/disable-lro-gro.service
   ```
2. Add the following content:
   ```
   [Unit]
   Description=Disable LRO and GRO

   [Service]
   Type=oneshot
   ExecStart=/sbin/ethtool -K <network_adapter> gro off
   ExecStart=/sbin/ethtool -K <network_adapter> lro off

   [Install]
   WantedBy=multi-user.target
   ```
3. Enable the service:
   ```bash
   sudo systemctl enable disable-lro-gro.service
   ```

---

### **4. Install Snort 3**
1. Download and install Snort 3:
   ```bash
   cd ~/snort
   wget https://github.com/snort3/snort3/archive/refs/tags/3.1.74.0.tar.gz -O snort3-3.1.74.0.tar.gz
   tar -xzvf snort3-3.1.74.0.tar.gz
   cd snort3-3.1.74.0
   ./configure_cmake.sh --prefix=/usr/local --enable-jemalloc
   cd build
   make
   sudo make install
   ```

2. Update shared libraries:
   ```bash
   sudo ldconfig
   ```

---

### **5. Install PulledPork**
1. Clone the PulledPork repository:
   ```bash
   git clone https://github.com/shirkdog/pulledpork3.git
   cd ~/snort/pulledpork3
   ```
2. Install PulledPork:
   ```bash
   sudo mkdir /usr/local/bin/pulledpork3
   sudo cp pulledpork.py /usr/local/bin/pulledpork3
   sudo cp -r lib/ /usr/local/bin/pulledpork3
   sudo chmod +x /usr/local/bin/pulledpork3/pulledpork.py
   sudo mkdir /usr/local/etc/pulledpork3
   sudo cp etc/pulledpork.conf /usr/local/etc/pulledpork3
   ```

---

### **6. Verification and Testing**
1. Verify Snort installation:
   ```bash
   snort -V
   ```
   - Confirms the installed version of Snort.

2. Test PulledPork functionality:
   ```bash
   /usr/local/bin/pulledpork3/pulledpork.py -c /usr/local/etc/pulledpork3/pulledpork.conf
   ```
   - Ensures PulledPork is properly configured and functional.

---


