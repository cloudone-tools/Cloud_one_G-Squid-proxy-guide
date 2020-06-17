## Introduction:
=================

This is a quick start guideline to how to implement deep security agent SaaS
with Squid proxy in AWS.

This document is divided into two sections:

-   ### Setup your environment:

 This section contains how to setup your environment and walkthrough in every
    step using AWS.

 ![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image1.jpg)
    

-   ### Install deep security agent:

 This section contains a guideline for how to install deep security agent in
    cloud on workload security dashboard.

### Setup your enviroment
=========================

1.  #### Virtual private cloud (VPC):

-   Create VPC and assign any private IP address ranges:

    ![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image2.jpg)

>Subnet sizing for private IPv4 in AWS:
>
>| **IP ranges**                   | **Prefix**          |
>|---------------------------------|---------------------|
>| 10.0. 0.0 - 10.255. 255.255     | (10/8 prefix)       |
>| 172.16. 0.0 - 172.31. 255.255   | (172.16/12 prefix)  |
>| 192.168. 0.0 - 192.168. 255.255 | (192.168/16 prefix) |
>
2.  #### Internet gateway:

-   Internet gateway is acting as bridge connection between internet and VPC.

![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image3.jpg)

After you create internet gateway, you need to associate it with VPC.

3.  #### Subnets:
>
> -   you need to create two subnets which are:
>
  >> **Public Subnet:**
>>
>>  It will configure to access the internet through the internet gateway if the traffic is going to the internet & this subnet will
>> be used for Squid proxy instance.
>>
>>![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/Capture2.JPG)
>>
>> **Private Subnet:**
>>
>>![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/Capture.JPG)
>> - It will configure to access to internet through Squid proxy if the traffic is
>>going to the internet.

4.  #### Squid proxy:

-   You need to connect to proxy server ec2-user running on Linux OS by using
    putty:

>   Open putty On session section –\<Write (username \@Public IP address) and
>  choose on connection type SSH.
>
>   ![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image6.jpg)
>
>   Open Puttygen program then choose load then select key pair file that you
>  assign it with this ec2 instance then save private key.
>
> ![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image7.jpg)
>
>   On putty interface choose SSH section then Auth browse your private key file
>  that you saved from Puttygen then click open to connect ec2-instance.
>
>   ![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image8.jpg)
>
>   Squid installation commands in Linux terminal: 
>
>   sudo yum install -y squid ((to install squid proxy))
>   
>   sudo service squid start ((To start squid proxy Server))
>
>   sudo chkconfig squid on ((to turn on  squid proxy ))
>
>   sudo vim /etc/squid/squid.conf (( Edit the SQUID Configurations file to
>   allow TrendMicro and AWS domains only))
>
>   add the following commands in squid.conf:
>>    **Restrict TrendMicro Access**
>>
>>  **acl GOOD dstdomain .trendmicro.com .amazonaws.com**
>>
>>  **http_access allow GOOD**
>>
>>  **http_access deny all**
>>  ![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image9.jpg)
>>  ![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image10.jpg)
>>
>>  After you add these commands Press Esc key and type: wq to save changes to
>> a file and exit from vim.
>>
>>   sudo service squid restart(( restart Squid proxy.))

5.  #### Route tables:

> -   You need to create two route tables which are:
>
>>-   **Private route table:**
>>
>>  Once you create private route table, the default route will navigate local
>>  traffic through local network and it needs to add new route which navigate any
>>  traffic to go through Squid proxy if the traffic is going to internet (you
>>  need to associate this route within private subnet).
>>
>>   ![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image5.jpg)
>>
>>-   **Public route table:**
>>
>>  Once you create public route table, the default route will navigate local
>>  traffic to go through local network and it needs to add new route which
>>  navigate any traffic to go through internet gateway if the traffic is going
>>  to internet (you need to associate this route within public subnet).
>>    ![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image4.jpg)
>>
>>
6.  #### Security Group:

>-   Aws security group let you limit and control inbound & outbound traffic in
>    EC2 instances level.
>
>>-   **Security group Squid proxy in public subnet**:
>>
>>  **Inbound SSH:** Allowing SSH traffic to accept inbound traffic from authorized
>>  external device to squid proxy, (**Note:** specify your own public IP to
>>  prevent an authorized connection to squid proxy (optional)).
>>  
>>  **Inbound squid proxy:** allowing 3128 port traffic from internet to squid proxy.
>>    ![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image13.jpg)
>>
>>  **Outbound HTTPS:** Allowing HTTPS outbound traffic and it is used for various
>>  Deep Security cloud services, you can restrict IP addresses in HTTPS port
>> ,for more info please check this
>> URL:[https://help.deepsecurity.trendmicro.com/10/0/Manage-Components/ports.html\#Deep2]
>>(https://help.deepsecurity.trendmicro.com/10/0/Manage-Components/ports.html%23Deep2)
>>    ![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image14.jpg)
>>
>>-   **Security group in private subnet:**
>>
>>   **Inbound squid proxy:** allowing 3128 port traffic from squid proxy to Deep
>>    security agent that has been installed in ec2 instance.
>>
>>![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image15.jpg)
>>
>>  **Outbound squid proxy:** allowing 3128 port traffic from ec2 instance that has been
>>installed Deep security agent to squid proxy server.
>> ![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image16.jpg)
>>
>>**Note:**you need to assign private IP squid proxy in inbound/outbound ec2
>>instances in private subnet.

**Install deep security agent**
===============================

1.  **Signup/Login Dashboard:**

![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image17.jpg)

You can sign up or login by using this URL: <https://cloudone.trendmicro.com/>

![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image18.jpg)

After successfully registering, login your account, you will see cloud one
dashboard click on workload security.

2.  **Add AWS account:**

Cloud one workload security dashboard will be opened click on computers tab
then on left side right click on computers then choose add AWS account.

   ![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image19.jpg)

3.  **Setup type:**

You can add AWS account by using quick & Advanced.

   ![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image20.jpg)

4.  **Add squid proxy server in workload security:**

![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image21.jpg)

Go to administration then click system setting, click new to add new proxy.
>
> In new proxy properties interface, write the private IP squid proxy with
>port.
>
> ![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image22.jpg)
>
5.  **Deploy deep security agent on server:**

In cloud one workload dashboard, click on support and you will see two ways of
deployment.
![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image23.jpg)
>
>-   **Download agents:**
>
>    If you want to download Agent package installer and install it manually.
>   **Note**: installation guide for manually install deep security
>    agent:<https://help.deepsecurity.trendmicro.com/10_2/aws/Get-Started/Install/install-dsa.html>
>
>    ![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image24.jpg)
>
>-   **Deployment script:**
>
>    This option will install agent by using script.
>
>    Installation guide for install deep security agent using deployment script:
>    <https://help.deepsecurity.trendmicro.com/10_2/aws/Add-Computers/ug-add-dep-scripts.html>
>
>>- In proxy to contact deep security manager & proxy to contact relay options:
>>
>>![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image25.jpg)
>>  you need to select squid proxy that you already added in previous step.

6.  **Check the state of deep security agent after deployment:**

-   Go to computer and see the status of instance.

![](https://github.com/OmarR00t/documentation-beta-/blob/master/Squid%20proxy_screenshots/image26.jpg)
