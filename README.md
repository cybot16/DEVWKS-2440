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
