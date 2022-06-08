# CLive2022_DEVWKS-3240

# Lab Introduction
To access the lab, you will need to SSH to the VM specific host. From the VM host you will have access to the switch and the remaining software dependencies for the lab. Please find below the actual lab environment and instructions. 


# Lab environment
![](Clive3240_env.png)


# Accessing the lab environment 
Identify your pod# and log into your respective pod# using SSH:

Note: Use your pod number in the # symbol for the SSH command

ssh -L 15152:10.1.1.3:3000 -L 15151:10.1.1.3:8480 auto@pod#-xelab

Password: programmability@LAB


