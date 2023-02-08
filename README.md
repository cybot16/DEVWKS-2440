# DEVWKS-2440

## 1 - Login to dcloud:
#### a - Discover the topology
#### b - Connect to the anyconnect dcloud VPN
#### c - Connect to CML
#### d - Connect to Centos server using ssh

## 2 - Login to nso cli:
       [cisco@centos ~]$ ncs_cli -C
#### a - Show the onboarded devices on NSO:
      cisco@ncs# show devices list
      NAME    ADDRESS     DESCRIPTION  NED ID   ADMIN STATE
      -----------------------------------------------------
      asr9k1  198.18.1.2  -            netconf  unlocked
#### b - Test device connectivity:
      cisco@ncs# devices device asr9k1 connect
      result true
      info (cisco) Connected to asr9k1 - 198.18.1.2:830
#### c - Sync-from the device:
      cisco@ncs# devices device asr9k1 sync-from
      result true
#### c - Activate devtools:      
      cisco@ncs# devtools true
#### d - Entering configuration mode:
      cisco@ncs# config terminal
      Entering configuration mode terminal
      cisco@ncs(config)#
#### e - creating a new netconf-ned-builder project:
      cisco@ncs(config)# netconf-ned-builder project cisco-iosxr 7.5 device asr9k1 vendor cisco local-user cisco
      cisco@ncs(config-project-cisco-iosxr/7.5)# commit
      Commit complete.
      cisco@ncs(config-project-cisco-iosxr/7.5)# end
      cisco@ncs#
 #### f - Fetch module list:
      cisco@ncs# netconf-ned-builder project cisco-iosxr 7.5 fetch-module-list
      cisco@ncs#

 #### g - Selecting the desired modules:
      cisco@ncs# netconf-ned-builder project cisco-iosxr 7.5 module Cisco-IOS-XR-* * select
      cisco@ncs#
 
 #### h - Verifying selected modules:
      cisco@ncs# show netconf-ned-builder project cisco-iosxr 7.5 module Cisco-IOS-XR-ifmgr-cfg 2020-10-01
      module Cisco-IOS-XR-ifmgr-cfg 2020-10-01
       namespace http://cisco.com/ns/yang/Cisco-IOS-XR-ifmgr-cfg
       import Cisco-IOS-XR-types
       import cisco-semver
       location  [ NETCONF ]
       status    selected,downloaded
       
  #### i - Building the NED:     
       cisco@ncs# netconf-ned-builder project cisco-iosxr 7.5 build-ned
       cisco@ncs#
  #### j - Verifying Build:     
       cisco@ncs# show netconf-ned-builder project cisco-iosxr 7.5 build-status
       build-status success
       cisco@ncs#
  #### k - Verifying the creation of the new NED:    
       cisco@ncs#exit
       [cisco@centos ~]$ ls /var/opt/ncs/state/netconf-ned-builder
       cache  cisco-iosxr-nc-7.5
       [cisco@centos ~]$
  #### l - Installing the new NED:    
       [cisco@centos ~]$ cp -avr /var/opt/ncs/state/netconf-ned-builder/cisco-iosxr-nc-7.5/ /var/opt/ncs/packages/
  #### m - Updating the CDB schema: 
       [cisco@centos ~]$ ncs_cli -C
       cisco@ncs# packages reload
       cisco@ncs# packages reload
       >>> System upgrade is starting.
       >>> Sessions in configure mode must exit to operational mode.
       >>> No configuration changes can be performed until upgrade has completed.
       >>> System upgrade has completed successfully.
       reload-result {
           package cisco-iosxr-nc-7.5
           result true
       }
       cisco@ncs#
       
  #### n - Migrating the device to the new NED:
         cisco@ncs# devices device asr9k1 migrate new-ned-id cisco-iosxr-nc-7.5
         cisco@ncs#
  
  #### o - Testing device connectivity and sync-from:
       cisco@ncs# devices device asr9k1 connect
       result true
       info (cisco) Connected to asr9k1 - 198.18.1.2:830
       cisco@ncs# devices device asr9k1 sync-from
       result true
       cisco@ncs#
       
  #### p - Checking if the CDB is populated with device config:
       cisco@ncs# show running-config devices device asr9k1 config interface-configurations interface-configuration act GigabitEthernet0/0/0/0
       devices device asr9k1
        config
         interface-configurations interface-configuration act GigabitEthernet0/0/0/0
          shutdown
         !
        !
       !
   #### p - Testing configuring the device using the new NED:
       cisco@ncs(config)# devices device asr9k1 config interface-configurations interface-configuration act GigabitEthernet0/0/0/0 description DEVWKS-2440
       cisco@ncs(config-interface-configuration-act/GigabitEthernet0/0/0/0)# commit dry-run outformat native
       native {
           device {
               name asr9k1
               data <rpc xmlns="urn:ietf:params:xml:ns:netconf:base:1.0"
                         message-id="1">
                      <edit-config xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0">
                        <target>
                          <candidate/>
                        </target>
                        <test-option>test-then-set</test-option>
                        <error-option>rollback-on-error</error-option>
                        <config>
                          <interface-configurations xmlns="http://cisco.com/ns/yang/Cisco-IOS-XR-ifmgr-cfg">
                            <interface-configuration>
                              <active>act</active>
                              <interface-name>GigabitEthernet0/0/0/0</interface-name>
                              <description>DEVWKS-2440</description>
                            </interface-configuration>
                          </interface-configurations>
                        </config>
                      </edit-config>
                    </rpc>
           }
       }
       cisco@ncs(config-interface-configuration-act/GigabitEthernet0/0/0/0)#commit
       Commit complete.
       cisco@ncs(config-interface-configuration-act/GigabitEthernet0/0/0/0)#
       
  #### c - Verifying config on device:    
       RP/0/RP0/CPU0:XR-9k#show running-config interface GigabitEthernet 0/0/0/0
       interface GigabitEthernet0/0/0/0
        description DEVWKS-2440
        shutdown
       !


## 3 - Create a service 
#### a - Go to packages directory:
       [cisco@centos ~]$ cd /var/opt/ncs/packages/
#### b - Make a new service package:
       [cisco@centos packages]$ ncs-make-package --service-skeleton python-and-template interface-config-nc
#### c - Generating the config template:
       <config>
         <interface-configurations xmlns="http://cisco.com/ns/yang/Cisco-IOS-XR-ifmgr-cfg">
           <interface-configuration>
             <active>act</active>
             <interface-name>GigabitEthernet0/0/0/0</interface-name>
             <ipv4-network xmlns="http://cisco.com/ns/yang/Cisco-IOS-XR-ipv4-io-cfg">
               <addresses>
                 <primary>
                   <address>10.0.0.1</address>
                   <netmask>255.255.255.252</netmask>
                 </primary>
               </addresses>
             </ipv4-network>
           </interface-configuration>
         </interface-configurations>
       </config>
#### d - Replacing the service template with the generated template and replace data with variables:
       [cisco@centos templates]$ vi /var/opt/ncs/packages/interface-config-nc/templates/interface-config-nc-template.xml
       
             <config-template xmlns="http://tail-f.com/ns/config/1.0">
                <devices xmlns="http://tail-f.com/ns/ncs">
                  <device>
                    <name>{/device}</name>
                    <config>
                     <interface-configurations xmlns="http://cisco.com/ns/yang/Cisco-IOS-XR-ifmgr-cfg">
                      <interface-configuration>
                      <active>act</active>
                      <interface-name>{/interface-name}</interface-name>
                      <ipv4-network xmlns="http://cisco.com/ns/yang/Cisco-IOS-XR-ipv4-io-cfg">
                        <addresses>
                         <primary>
                          <address>{/ip-address}</address>
                          <netmask>{/netmask}</netmask>
                         </primary>
                        </addresses>
                       </ipv4-network>
                      </interface-configuration>
                     </interface-configurations>
                    </config>
                  </device>
                </devices>
              </config-template>
       
