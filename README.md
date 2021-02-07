__<h1>How to deploy Apache Httpd webserver on Docker using Ansible</h1>__

![Ansible_Docker_HTTPD](https://cdn-images-1.medium.com/max/1000/1*z4ypa_pY7UksT1Up9HPB6w.jpeg)<br>

<h2> Content </h2>
<h4><ul>
<li><i>About Ansible</i></li>
<li><i>About Docker</i></li>
<li><i>About Apache HTTPD Webserver</i></li> 
<li><i>Project Understanding</i></li> 
</ul></h4><br>

<h2>About Ansible</h2>
<h3>What is Ansible ?</h3>
<i><b>Ansible</b> is a radically simple IT automation engine that automates cloud provisioning, configuration management, application deployment, intra-service orchestration, and many other IT needs.</i>

<h3>Why use Ansible ?</h3>
<h4><ol>
<li>
  Simple
  <ul>
    <li><i>Human readable automation</i></li>
    <li><i>No special coding skills needed</i></li>
    <li><i>Tasks executed in order</i></li>
    <li><i>Get productive quickly</i></li>
  </ul>
</li><br>
<li>
  Powerful
  <ul>
    <li><i>App deployment</i></li>
    <li><i>Configuration management</i></li>
    <li><i>Workflow orchestration</i></li>
    <li><i>Orchestrate the app lifecycle</i></li>
  </ul>
</li><br>
<li>
  Agentless
  <ul>
    <li><i>Agentless architecture</i></li>
    <li><i>Uses OpenSSH and WinRM</i></li>
    <li><i>No agents to exploit or update</i></li>
    <li><i>Predictable, reliable and secure</i></li>
  </ul>
</li><br>
</ol></h4>
For Ansible Documentation, visit the link mentioned below:<br>
https://docs.ansible.com/

<h2>About Docker</h2>
<ul>
    <li><i><b>Docker</b> is a open source platform for building, deploying and managing containerized applications. It enables developer to package applications into containers.</i></li>
    <li><i><b>Containers</b> are the executable components that combines the application source code with the Operating System(OS) libraries and dependencies required to run the code in any environment.</i></li>
    <li><i>Docker makes it easier, simpler and safer to build, deploy and manage containers</i></li>
</ul>
For Docker Documentation, visit the link mentioned below:<br>
https://docs.docker.com/

<h2>About Apache HTTPD Webserver</h2>
<ul>
  <li><i>The <b>Apache HTTP Server</b>, colloquially called <b>Apache</b>, is a free and open-source cross-platform web server software, released under the terms of <b>Apache License 2.0</b>.</i></li>
  <li><i>Apache is developed and maintained by an open community of developers under the auspices of the <b>Apache Software Foundation</b>.</i></li>
  <li><i>The vast majority of Apache HTTP Server instances run on a Linux distribution, but current versions also run on Microsoft Windows, OpenVMS and a wide variety of Unix-like systems.</i></li>
</ul>

For Apache HTTPD Documentation, visit the link mentioned below:<br>
https://httpd.apache.org/docs/

<h2>Project Understanding</h2>
<p>Let's understand this implementation part by part</p>
 
 <h3>Part 1 : Ansible Playbook (main.yml)</h3>
 Let's understand it task by task:<br><br>
 
 <ul>
   <li>"<i>Mount Directory Creation</i>" : Creation of Mount Directory</li><br>
   
   ```
   - name: Mount Directory Creation
    file:
      state: directory
      path: "{{ mount_directory }}"
   ```
   <br>
   <li>"<i>Mount Directory to CDROM</i>" : Mount the directory created in previous task to the CDROM</li><br>
   
   ```
   - name: Mount Directory to CDROM
    mount:
      src: "/dev/cdrom"
      path: "{{ mount_directory }}"
      state: mounted
      fstype: "iso9660"
   ```
   <br>
   <li>"<i>YUM Repository - AppStream</i>" : Addition of AppStream YUM repository</li><br>
   
   ```
   - name: YUM Repository - AppStream
    yum_repository:
      baseurl: "{{ mount_directory }}/AppStream"
      name: "DVD1"
      description: "YUM Repository - AppStream"
      enabled: true
      gpgcheck: no
   ```
   <br>
   <li>"<i>YUM Repository - BaseOS</i>" : Addition of BaseOS YUM repository</li><br>
   
   ```
   - name: YUM Repository - BaseOS
    yum_repository:
      baseurl: "{{ mount_directory }}/BaseOS"
      name: "DVD2"
      description: "YUM Repository - BaseOS"
      enabled: true
      gpgcheck: no
   ```
   <br>
   <li>"<i>Docker Repository Setup</i>" : Creation of repository for Docker tor its installation</li><br>
   
   ```
   - name: Docker Repository Setup
    yum_repository:
      baseurl: https://download.docker.com/linux/centos/7/x86_64/stable/
      name: "docker"
      description: "Docker repo"
      enabled: true
      gpgcheck: no
   ```
   <br>
   <li>"<i>Setting pip installer</i>" : Setting up pip installer to install Docker SDK for Python</li><br>
   
   ```
   - name: Setting pip installer
    package:
      name: "python36"
      state: present
   ```
   <br>
   <li>"<i>Setting SELinux Permissive</i>" : Disabling SELinux for successful creation of Docker container</li><br>
   
   ```
   - name: Setting SELinux Permissive
    selinux:
      policy: targeted      
      state: permissive
   ```
   <br>
   <li>"<i>Check whether Docker is installed or not</i>" : Checks for presence of Docker by using the command "rpm -q docker-ce" and is used to make Docker Installation idempotent</li><br>
   
   ```
   - name: Check whether Docker is installed or not
    command: "rpm -q docker-ce"
    register: docker_check
    ignore_errors: yes
   ```
   <br>
   <li>"<i>Docker Installation</i>" : Installs Docker only when the above task satisfies the when condition</li><br>
   
   ```
   - name: Docker Installation
    command: "yum install docker-ce --nobest -y"
    when: '"is not installed" in docker_check.stdout'
   ```
   <br>
   <li>"<i>Starting Docker Service</i>" : Starts Docker service</li><br>
   
   ```
   - name: Starting Docker Service
    systemd:
      name: "docker"
      state: started
   ```
   <br>
   <li>"<i>Stopping Firewall Service</i>" : Disabling Firewall for accessing web page from the browser present outside the VM(Virtual Machine)</li><br>
   
   ```
   - name: Stopping Firewall Service
    systemd:
      name: "firewalld"
      state: stopped
   ```
   <br>
   <li>"<i>Installation of Docker SDK for Python for using Docker modules in Ansible</i>" : Installation of Docker SDK for Python i.e., "docker-py' for using Docker modules in Ansible</li><br>
   
   ```
   - name: Installation of Docker SDK for Python for using Docker modules in Ansible
    pip:
      name: "docker-py"
   ```
   <br>
   <li>"<i>Creation of directory</i>" : Creation of directory to be mounted with the HTTPD's document root within Docker container</li><br>
   
   ```
   - name: Creation of directory
    file:
      path: /root/web_page/
      state: directory
   ```
   <br>
   <li>"<i>Copying the web page to the directory created</i>" : Web page present within web directory(more details below) is copied to the directory created in previous tasks</li><br>
   
   ```
   - name: Copying the web page to the directory created
    copy:
      src: web/index.html
      dest: /root/web_page/
   ```
   <br>
   <li>"<i>Restarting Docker Service</i>" : Docker Service is restarted to resolve the error mentioned below</li><br>
   
   ```
   - name: Restarting Docker Service
    systemd:
      name: "docker"
      state: restarted
   ```
   ![External_Connectivity_Error](https://cdn-images-1.medium.com/max/1000/1*g_3mwBbqg-wtyc-sXk47gQ.png)<br>

   <br>
   <li>"<i>Creation of Docker container for httpd</i>" : Involves creation of Docker container using httpd image, the directory created in previous tasks is mounted to the Document Root of httpd and PAT(Port Address Translation) is performed to expose the container created</li><br>
   
   ```
   - name: Creation of Docker container for httpd
    docker_container:
      name: httpd_container
      image: httpd
      pull: yes
      detach: yes
      state: started
      ports:
      - "{{ port_number }}:80"
      volumes: 
      - /root/web_page/:/usr/local/apache2/htdocs/
   ```
   <br>
 </ul>
 
 <h3>Part 2 : Variable File (vars.yml)</h3><br>
 
 ```
 mount_directory: "/dvd1"
 port_number: "82"
 ```
  
 Used to make Playbook more dynamic and it specifies the value of Port Number(for PAT) and Mount Directory
 <br>
 
 <h3>Part 3 : Web  Page</h3><br>
 
 ```
 Hello, it's working
 ```
 
 It specifies the web page to be accessed from outside the container
 <br>
 
 <h3>Part 4 : Implementation & Output</h3><br>
 <ul>
   <li><i>Implementation</i></li><br>
   
   ![Playbook_Execution](/execution/execution.gif)<br>
   
   <li><i>Output</i></li><br>
   
   ![Output](https://miro.medium.com/max/875/1*C3bgmnyYS1aD-mTHMK5TOQ.png)<br>
 </ul><br>
 
 <h2>Thank You :smiley:<h2>
 <h3>LinkedIn Profile</h3>
 https://www.linkedin.com/in/satyam-singh-95a266182
 
 
 
