# CLive2022_DEVWKS-3240
## Version 17.7

# Lab Introduction
To access the lab, you will need to SSH to the VM specific host. From the VM host you will have access to the switch and the remaining software dependencies for the lab. Please find below the actual lab environment and instructions. 


# Lab environment
![](Clive3240_env.png)


# Accessing the lab environment 
Identify your pod# and log into your respective pod# using SSH:

Note: Use your pod number in the # symbol for the SSH command

ssh -L 15152:10.1.1.3:3000 -L 15151:10.1.1.3:8480 auto@pod#-xelab

Password: programmability@LAB

Once you logged into the VM, you should be able to see fhe following prompt:
![](logged_vm.png)



# Telnet into the Catalyst 9300
From the VM prompt, enter the following commands to Telnet into the Catalyst 9300. We have DNS naming configured so we wont use an IP address but an actual name. 

auto@pod#-xelab:~$ telnet c9300

user: admin

Password: Cisco123


# Configuring Telemetry Subscriptions on the Catalyst 9300
1-Every process that you need to monitor from the device requires a subscription. We will create for subscriptions to monitor the following aspects: CPU, Power, Memory and Temperature.

2-Configure the type of encoding, in our case is: ‘encode-kvgpb’

3-YANG Push can be used to monitor configuration or operational datastore changes. We will use: ‘ stream yang-push’ 

4-Periodicity. Specify how frequently you want to send the traffic (in milliseconds) and the receiver of the traffic.

5-Include the receiver of the traffic, in this case it is the switch: 10.1.1.5. 

6-Copy&paste or enter the following commands, exactly as they appear on the Catalyst 9300:


configure terminal

telemetry ietf subscription 1010

 encoding encode-kvgpb
 
 filter xpath /process-cpu-ios-xe-oper:cpu-usage/cpu-utilization/five-seconds
 
 source-address 10.1.1.5
 
 stream yang-push
 
 update-policy periodic 2000
 
 receiver ip address 10.1.1.3 57500 protocol grpc-tcp



telemetry ietf subscription 1020

 encoding encode-kvgpb
 
 filter xpath /poe-ios-xe-oper:poe-oper-data
 
 source-address 10.1.1.5
 
 stream yang-push
 
 update-policy periodic 2000
 
 receiver ip address 10.1.1.3 57500 protocol grpc-tcp
 


telemetry ietf subscription 1030

  encoding encode-kvgpb
 
  filter xpath /memory-ios-xe-oper:memory-statistics/memory-statistic
 
  source-address 10.1.1.5
 
  stream yang-push
 
  update-policy periodic 2000
 
  receiver ip address 10.1.1.3 57500 protocol grpc-tcp
 


telemetry ietf subscription 1040

 encoding encode-kvgpb
 
 filter xpath /oc-platform:components/component/state/temperature
 
 source-address 10.1.1.5
 
 stream yang-push
 
 update-policy periodic 2000
 
 receiver ip address 10.1.1.3 57500 protocol grpc-tcp
 ![](mdt_subscriptions.png)
 
 
 
 # Increased Observability with IOS XE Telemetry displayed in Grafana
![](grafana_dashboard.png)
Grafana is an open source solution for running data analytics, pulling up metrics selective metrics out of the huge amount of data that we are monitoring daily from our devices and apps with the help of customizable dashboards.

Grafana connects with every possible data source or databases such as Graphite, Prometheus, Influx DB, ElasticSearch, MySQL, PostgreSQL etc. In this case we will extract the information the subscriptions that were created on the switch. This data has been  sent from the switch to Influx DB. Now we will display that into Grafana dashboards.

Grafana being an open source solution also enables us to write plugins from scratch for integration with several different data sources.

## Open the Grafana dashboard 
Open http://localhost:15152/

Username: admin 

Password: Cisco123


# Enhanced security via certificates (gNOI) for Model-Driven Telemetry

## gNOI Certificate Management Client

A simple shell binary that performs Certificate Management client operations against a gNOI Target.

## Certificates

Only the Root certificate and private key are required for this client. The client will:

* generate a client certificate for establishing the connection to the Target

* sign target signing requests for installing or rotating certificates on the Target

The client certificates can also be provided to establish the connection to the target and will be used instead.

For the sake of brevity, we will just take care of the aspects of this configuration: 1) the GNXI switch configuration and the certificate provision on the VM. 


## 1-GNXI configuration on the Catalyst 9300

C9300#conf t

Enter configuration commands, one per line.  End with CNTL/Z.

C9300(config)#gnxi

C9300(config)#gnxi secure-init

C9300(config)#gnxi secure-server

C9300(config)#gnxi secure-port 9339

After you entered these commands, you we will see the self-signed option on the switch 
C9300#show gnxi state detail 



## 2-Provision the certificates on the Virtual Machine

Enter the gnmi_ssl 

Copy and paste the following command exactly as it is on the Pod# VM

../../gnoi_cert -target_addr c9300:9339 -op provision -target_name c9300 -alsologtostderr -organization "jcohoe org" -ip_address 10.1.1.5 -time_out=10s -min_key_size=2048 -cert_id mdt_cert -state BC -country CA -ca ./rootCA.pem -key ./rootCA.key

This is going to install the certificate on the switch with the name that was specified (mdt_cert)

Verify the switch log to see the cert name was created

Check the gnxi state details to validate it changed. (show gnxi state detail )


## Add Security. 

* Provision the certificate on the VM 

* Configure the switch to use the certificates that were installed on the VM.

* Verify the certificates are in use now.




