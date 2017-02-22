Core Admin Competencies

Skeleton:

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues


Administration (?)


Authentication

	General

		- Documentation

		http://docs.alfresco.com/5.1/concepts/auth-subsystem-types.html (chart for auth/file servers matrix)

	Alfresco NTLM

		- Documentation

			http://docs.alfresco.com/5.1/concepts/auth-alfrescontlm-intro.html (start)
		http://docs.alfresco.com/5.1/concepts/auth-alfrescontlm-ntlm.html (some details about Alfresco's NTLM):

			The alfrescoNtlm subsystem supports optional NTLM Single Sign-On (SSO) functions for WebDAV.

			Note:
			NTLM v2 is supported, which is more secure that the NTLM v1. If the client does not support NTLMv2, it will automatically downgrade to NTLMv1.
			By using NTLM authentication to access Alfresco WebDAV sites, the web browser can automatically log in.

			When SSO is enabled, Internet Explorer will use your Windows login credentials when requested by the web server. Firefox and Mozilla also support the use of NTLM but you need to add the URI to the Alfresco site that you want to access to network.automatic-ntlm-auth.trusted-uris option (available through writing about:config in the URL field) to allow the browser to use your current credentials for login purposes.

			The Opera web browser does not support NTLM authentication. The browser is detected and will be sent to the usual Alfresco logon page.

			In this configuration, Alfresco must still store its own copy of your MD4 password hash. In order to remove this need and authenticate directly with a Windows domain controller, consider using the pass-through subsystem.

		http://docs.alfresco.com/5.1/concepts/auth-alfrescontlm-props.html (NTLM properties):

			ntlm.authentication.sso.enabled
				A Boolean that when true enables NTLM based Single Sign On (SSO) functionality in the Web clients. When false and no other members of the authentication chain support SSO, password-based login will be used.

			ntlm.authentication.sso.fallback.enabled
				If SSO fails, a fallback authentication mechanism is used. The default value is true.
			
			ntlm.authentication.mapUnknownUserToGuest
				Specifies whether unknown users are automatically logged on as the Alfresco guest user during Single Sign-On (SSO).
			
			alfresco.authentication.authenticateCIFS
				A Boolean that when true enables Alfresco-internal authentication for the CIFS server. When false and no other members of the authentication chain support CIFS authentication, the CIFS server will be disabled.
			
			alfresco.authentication.allowGuestLogin
				Specifies whether to allow guest access to Alfresco.

		- Supported Platforms and Notes

			Supported widely on all platforms.

		- Downloads (if applicable)

			N/A

		- Setup / Recipes

			Set up NTLM SSO:

			Alfresco Share exists as a separate web application to the main Alfresco repository WAR file. It can run in the same application server instance on the same machine as the main web application, or it can run on a completely separate application server instance on a different machine. Share uses HTTP(S) to communicate with the configured Alfresco repository.

			Locate the following configuration file:
				<web-extension>\share-config-custom.xml

			Edit the file, and then uncomment the following section:
 			<!-- 
        		SSO authentication config for Share
        		NOTE: change localhost:8080 below to appropriate alfresco server location if required
   			-->

   			<config evaluator="string-compare" condition="Remote">
		      <remote>
		         <connector>
		            <id>alfrescoCookie</id>
		            <name>Alfresco Connector</name>
		            <description>Connects to an Alfresco instance using cookie-based authentication</description>
		            <class>org.alfresco.web.site.servlet.SlingshotAlfrescoConnector</class>
		         </connector>
		         
		         <endpoint>
		            <id>alfresco</id>
		            <name>Alfresco - user access</name>
		            <description>Access to Alfresco Repository WebScripts that require user authentication</description>
		            <connector-id>alfrescoCookie</connector-id>
		            <endpoint-url>http://localhost:8080/alfresco/wcs</endpoint-url>
		            <identity>user</identity>
		            <external-auth>true</external-auth>
		         </endpoint>
		      </remote>
		   </config>

			From Alfresco One 5.1.2 onwards, the RSS feed URLs no longer use the alfresco endpoint. To make the RSS feed work, add one of the following to the above code.

			To use the RSS feeds authenticated through SSO, add the following:
			
			<endpoint>
				<id>alfresco-feed</id>
				<parent-id>alfresco</parent-id> 
				<name>Alfresco Feed</name>
				<description>Override Alfresco Feed - supports basic HTTP authentication via the EndPointProxyServlet</description> 
				<connector-id>alfrescoCookie</connector-id>
				<endpoint-url>http://localhost:8080/alfresco/wcs</endpoint-url>
				<identity>user</identity>
				<external-auth>true</external-auth>
			</endpoint>

			To use the RSS feeds with basic authentication ( or with older RSS feed readers) user, add the following:

			<endpoint>
				<id>alfresco-feed</id>
				<parent-id>alfresco</parent-id> 
				<name>Alfresco Feed</name>
				<description>Override Alfresco Feed - supports basic HTTP authentication via the EndPointProxyServlet</description> 
				<connector-id>alfresco</connector-id> 
				<endpoint-url>http://localhost:8080/alfresco/wcs</endpoint-url>
				<identity>user</identity>
				<basic-auth>true</basic-auth>
				<external-auth>false</external-auth>
			</endpoint>
			
			Change the <endpoint-url>http://localhost:8080/alfresco/wcs</endpoint-url> value to point to your Alfresco server location.
			Set the maxThreads option in the <TOMCAT_HOME>/conf/server.xml file:

			<Connector port="8080" protocol="HTTP/1.1" 
			               connectionTimeout="20000" 
			               redirectPort="8443" 
			               maxThreads="200" 
			 />

		- Debugging

			log4j.logger.org.alfresco.web.app.servlet.NTLMAuthenticationFilter=debug
			log4j.logger.org.alfresco.repo.webdav.auth.NTLMAuthenticationFilter=debug

		- Troubleshooting / Known Issues

			Note:
				If Share and Alfresco are installed on the same Tomcat, it is important to set the maxThreads option to 2*(expected number of concurrent requests). This is because each Share request spawns an Alfresco request.
				Restart Share.
				If you have configured alfrescoNtlm or passthru in your Alfresco authentication chain and enabled SSO, NTLM will be the active authentication mechanism.

			Note:
				SSO for RSS links only works if the user is first logged into Alfresco Share in the same browser session.

			Note:
				If you add extra administrator users in the authority-services-context.xml file and are using alfrescoNtlm, the extra users (other than the admin user) will no longer have administrator rights until you add them to the ALFRESCO_ADMINISTRATORS group.

			Issue:

				http://docs.alfresco.com/4.0/tasks/troubleshoot-ntlm.html

				Failure of NTLM logon on machines running Windows 7 or Internet Explorer 8.

					Troubleshooting

					This problem is most likely caused by enhanced security in Windows 7, Vista and Windows 2008. Previous versions of Windows (XP) would fall back to NTLM v1, if NTLM v2 failed.

					On Windows 7 clients, navigate to Control Panel > Administrative Tools > Local Security Policy.
					In the left pane, navigate to Security Settings > Local Policies > Security Options.
					In the right pane, find Network Security: LAN Manager authentication level.
					By default, the value of Network Security: LAN Manager authentication level is set to Send NTLMv2 response only. Refuse LM & NTLM.

					Set the value of Network Security: LAN Manager authentication level to Send LM and NTLM - use NTLMv2 session security if negotiated.
					This setting allows Windows 7 to use the more secure NTLM v2, if available, and fall back to NTLM v1 for Alfresco. If the machines are in a domain, it may be possible to change this setting on all of them via the group policy editor on the domain controller.


	LDAP-AD / Open LDAP

		- Documentation

			http://docs.alfresco.com/5.1/concepts/auth-ldap-intro.html (overview)
			http://docs.alfresco.com/5.1/concepts/auth-ldap-props.html (properties)
			http://docs.alfresco.com/5.1/tasks/auth-example-oneldap-ad.html (example of one ldap-ad domain)
			http://docs.alfresco.com/5.1/tasks/auth-example-twoldap-ad.html (multiple ldap-ad domain)

			http://docs.alfresco.com/5.1/concepts/sync-intro.html (synch intro)
			http://docs.alfresco.com/5.1/concepts/sync-triggers.html (synch triggers):

				Startup
					On system startup or restart of the Synchronization subsystem, a differential sync is triggered (unless disabled with configuration).

				Authentication
					On successful authentication of a user who does not yet exist locally, a differential sync is triggered (unless disabled with configuration).

				Schedule
					A scheduled job triggers synchronization in differential with removals mode every 24 hours. This can instead by scheduled in full mode if you set the synchronization.synchronizeChangesOnly property to false. The scheduling of this job can also be altered.

			http://docs.alfresco.com/5.1/concepts/sync-delete.html (sync delete rules / conditions)
			http://docs.alfresco.com/5.1/concepts/sync-collision.html (collision resolution)
			http://docs.alfresco.com/5.1/concepts/sync-props.html (synchronization properties)


		- Supported Platforms and Notes

			See https://www.alfresco.com/services/subscription/supported-platforms for PDFs

		- Downloads (if applicable)

			N/A

		- Setup / Recipes

			Use the following settings for ldap-ad auth and synch:

				### AD authentication only ###
				authentication.chain=alfrescoNtlm1:alfrescoNtlm,ldap-ad1:ldap-ad
				ldap.authentication.active=true
				ldap.authentication.allowGuestLogin=true
				ldap.authentication.userNameFormat=%s@example.foo
				ldap.authentication.java.naming.factory.initial=com.sun.jndi.ldap.LdapCtxFactory
				ldap.authentication.java.naming.provider.url=ldap://192.168.56.101:389
				ldap.authentication.java.naming.security.authentication=simple
				ldap.authentication.escapeCommasInBind=false
				ldap.authentication.escapeCommasInUid=false
				ldap.authentication.defaultAdministratorUserNames=Administrator

				ldap.synchronization.active=true
				ldap.synchronization.java.naming.security.principal=administrator@example.foo
				ldap.synchronization.java.naming.security.credentials=Alfr3sc0
				ldap.synchronization.queryBatchSize=1000
				ldap.synchronization.attributeBatchSize=1000
				synchronization.synchronizeChangesOnly=false
				synchronization.allowDeletions=true
				synchronization.syncWhenMissingPeopleLogIn=true

				ldap.synchronization.groupQuery=objectclass\=group
				ldap.synchronization.groupDifferentialQuery=(&(objectclass\=group)(!(modifyTimestamp<\={0})))

				ldap.synchronization.personQuery=(&(objectclass\=user)(userAccountControl\:1.2.840.113556.1.4.803\:\=512)(|(memberOf=cn\=AlfrescoAdmins,ou=alfresco,dc=example,dc=foo)(memberOf=cn\=AlfrescoUsers,ou=alfresco,dc=example,dc=foo)))

				ldap.synchronization.personDifferentialQuery=(&(objectclass\=user)(userAccountControl\:1.2.840.113556.1.4.803\:\=512)(|(memberOf=cn\=AlfrescoAdmins,ou=alfresco,dc=example,dc=foo)(memberOf=cn\=AlfrescoUsers,ou=alfresco,dc=example,dc=foo))(!(modifyTimestamp<\={0})))

				ldap.synchronization.groupSearchBase=ou\=alfresco,dc\=example,dc\=foo

				ldap.synchronization.userSearchBase=dc\=example,dc\=foo

				ldap.synchronization.modifyTimestampAttributeName=modifyTimestamp
				ldap.synchronization.timestampFormat=yyyyMMddHHmmss'.0Z'
				ldap.synchronization.userIdAttributeName=sAMAccountName
				ldap.synchronization.userFirstNameAttributeName=givenName
				ldap.synchronization.userLastNameAttributeName=sn
				ldap.synchronization.userEmailAttributeName=mail
				ldap.synchronization.userOrganizationalIdAttributeName=company
				ldap.synchronization.defaultHomeFolderProvider=largeHomeFolderProvider
				ldap.synchronization.groupIdAttributeName=cn
				ldap.synchronization.groupDisplayNameAttributeName=displayName
				ldap.synchronization.groupType=group
				ldap.synchronization.personType=user
				ldap.synchronization.groupMemberAttributeName=member
				ldap.synchronization.enableProgressEstimation=true


		- Debugging

			log4j.logger.org.alfresco.repo.security.authentication.ldap=debug
			log4j.logger.org.alfresco.repo.security.sync.ChainingUserRegistrySynchronizer=debug


			log4j.logger.org.alfresco.repo.security.sync=debug
			log4j.logger.org.alfresco.repo.security.person=debug
			log4j.logger.org.alfresco.repo.security.authentication=debug
			
			log4j.logger.org.alfresco.repo.importer.ImporterJob=debug
			log4j.logger.org.alfresco.repo.importer.ExportSourceImporter=debug
			
		- Troubleshooting / Known Issues

	
	Passthru

		- Documentation

			http://docs.alfresco.com/5.1/concepts/auth-passthru-intro.html (passthru intro)
			http://docs.alfresco.com/5.1/concepts/auth-passthru-domainprops.html (passthru domain level properties)
			http://docs.alfresco.com/5.1/concepts/auth-passthru-otherprops.html (other passthru properties)
			http://docs.alfresco.com/5.1/concepts/auth-passthru-domainmap.html (domain mappings)

		- Supported Platforms and Notes

			Windows only

		- Downloads (if applicable)

			N/A

		- Setup / Recipes

			Global properties example: 

			ldap.authentication.active=true
			ldap.authentication.allowGuestLogin=true
			ldap.authentication.userNameFormat=%s@example.foo
			ldap.authentication.java.naming.factory.initial=com.sun.jndi.ldap.LdapCtxFactory
			ldap.authentication.java.naming.provider.url=ldap://192.168.56.101:389
			ldap.authentication.java.naming.security.authentication=simple
			ldap.authentication.escapeCommasInBind=false
			ldap.authentication.escapeCommasInUid=false
			ldap.authentication.defaultAdministratorUserNames=Administrator

			ldap.synchronization.active=true
			ldap.synchronization.java.naming.security.principal=administrator@example.foo
			ldap.synchronization.java.naming.security.credentials=Alfr3sc0
			ldap.synchronization.queryBatchSize=1000
			ldap.synchronization.attributeBatchSize=1000
			synchronization.synchronizeChangesOnly=false
			synchronization.allowDeletions=true
			synchronization.syncWhenMissingPeopleLogIn=true

			ldap.synchronization.groupQuery=objectclass\=group
			ldap.synchronization.groupDifferentialQuery=(&(objectclass\=group)(!(modifyTimestamp<\={0})))

			ldap.synchronization.personQuery=(&(objectclass\=user)(userAccountControl\:1.2.840.113556.1.4.803\:\=512)(|(memberOf=cn\=AlfrescoAdmins,ou=alfresco,dc=example,dc=foo)(memberOf=cn\=AlfrescoUsers,ou=alfresco,dc=example,dc=foo)))

			ldap.synchronization.personDifferentialQuery=(&(objectclass\=user)(userAccountControl\:1.2.840.113556.1.4.803\:\=512)(|(memberOf=cn\=AlfrescoAdmins,ou=alfresco,dc=example,dc=foo)(memberOf=cn\=AlfrescoUsers,ou=alfresco,dc=example,dc=foo))(!(modifyTimestamp<\={0})))

			ldap.synchronization.groupSearchBase=ou\=alfresco,dc\=example,dc\=foo

			ldap.synchronization.userSearchBase=dc\=example,dc\=foo

			ldap.synchronization.modifyTimestampAttributeName=modifyTimestamp
			ldap.synchronization.timestampFormat=yyyyMMddHHmmss'.0Z'
			ldap.synchronization.userIdAttributeName=sAMAccountName
			ldap.synchronization.userFirstNameAttributeName=givenName
			ldap.synchronization.userLastNameAttributeName=sn
			ldap.synchronization.userEmailAttributeName=mail
			ldap.synchronization.userOrganizationalIdAttributeName=company
			ldap.synchronization.defaultHomeFolderProvider=largeHomeFolderProvider
			ldap.synchronization.groupIdAttributeName=cn
			ldap.synchronization.groupDisplayNameAttributeName=displayName
			ldap.synchronization.groupType=group
			ldap.synchronization.personType=user
			ldap.synchronization.groupMemberAttributeName=member
			ldap.synchronization.enableProgressEstimation=true


			ldap.synchronization.groupMemberAttributeName=member
			ldap.synchronization.enableProgressEstimation=true

			authentication.chain=passthru1:passthru,alfrescoNtlm1:alfrescoNtlm,ldap-ad1:ldap-ad
			ldap.authentication.active=false
			passthru.authentication.servers=EXAMPLE\\example.foo,example.foo 
			passthru.authentication.domain=
			passthru.authentication.useLocalServer=false
			passthru.authentication.guestAccess=false
			passthru.authentication.defaultAdministratorUserNames=Administrator
			passthru.authentication.authenticateFTP=true
			alfresco.authentication.authenticateCIFS=false
			ntlm.authentication.sso.enabled=true

			Share config settings:

			<alfresco-config>

				   <!-- Global config section -->
				   <config replace="true">
				      <flags>
				         <!--
				            Developer debugging setting to turn on DEBUG mode for client scripts in the browser
				         -->
				         <client-debug>false</client-debug>

				         <!--
				            LOGGING can always be toggled at runtime when in DEBUG mode (Ctrl, Ctrl, Shift, Shift).
				            This flag automatically activates logging on page load.
				         -->
				         <client-debug-autologging>false</client-debug-autologging>
				      </flags>
				   </config>
				   
				   <config evaluator="string-compare" condition="WebFramework">
				      <web-framework>
				         <!-- SpringSurf Autowire Runtime Settings -->
				         <!-- 
				              Developers can set mode to 'development' to disable; SpringSurf caches,
				              FreeMarker template caching and Rhino JavaScript compilation.
				         -->
				         <autowire>
				            <!-- Pick the mode: "production" or "development" -->
				            <mode>production</mode>
				         </autowire>

				         <!-- Allows extension modules with <auto-deploy> set to true to be automatically deployed -->
				         <module-deployment>
				            <mode>manual</mode>
				            <enable-auto-deploy-modules>true</enable-auto-deploy-modules>
				         </module-deployment>
				      </web-framework>
				   </config>

				   <!-- Disable the CSRF Token Filter -->
				   <!--
				   <config evaluator="string-compare" condition="CSRFPolicy" replace="true">
				      <filter/>
				   </config>
				   -->

				   <!--
				      To run the CSRF Token Filter behind 1 or more proxies that do not rewrite the Origin or Referere headers:

				      1. Copy the "CSRFPolicy" default config in share-security-config.xml and paste it into this file.
				      2. Replace the old config by setting the <config> element's "replace" attribute to "true" like below:
				         <config evaluator="string-compare" condition="CSRFPolicy" replace="true">
				      3. To every <action name="assertReferer"> element add the following child element
				         <param name="referer">http://www.proxy1.com/.*|http://www.proxy2.com/.*</param>
				      4. To every <action name="assertOrigin"> element add the following child element
				         <param name="origin">http://www.proxy1.com|http://www.proxy2.com</param>
				   -->

				   <!--
				      Remove the default wildcard setting and use instead a strict whitelist of the only domains that shall be allowed
				      to be used inside iframes (i.e. in the WebView dashlet on the dashboards)
				   -->
				   <!--
				   <config evaluator="string-compare" condition="IFramePolicy" replace="true">
				      <cross-domain>
				         <url>http://www.trusted-domain-1.com/</url>
				         <url>http://www.trusted-domain-2.com/</url>
				      </cross-domain>
				   </config>
				   -->

				   <!-- Turn off header that stops Share from being displayed in iframes on pages from other domains -->
				   <!--
				   <config evaluator="string-compare" condition="SecurityHeadersPolicy">
				      <headers>
				         <header>
				            <name>X-Frame-Options</name>
				            <enabled>false</enabled>
				         </header>
				      </headers>
				   </config>
				   -->

				   <!-- Prevent browser communication over HTTP (for HTTPS servers) -->
				   <!--
				   <config evaluator="string-compare" condition="SecurityHeadersPolicy">
				      <headers>
				         <header>
				            <name>Strict-Transport-Security</name>
				            <value>max-age=31536000</value>
				         </header>
				      </headers>
				   </config>
				   -->

				   <config evaluator="string-compare" condition="Replication">
				      <share-urls>
				         <!--
				            To discover a Repository Id, browse to the remote server's CMIS landing page at:
				              http://{server}:{port}/alfresco/service/cmis/index.html
				            The Repository Id field is found under the "CMIS Repository Information" expandable panel.

				            Example config entry:
				              <share-url repositoryId="622f9533-2a1e-48fe-af4e-ee9e41667ea4">http://new-york-office:8080/share/</share-url>
				         -->
				      </share-urls>
				   </config>

				   <!-- Document Library config section -->
				   <config evaluator="string-compare" condition="DocumentLibrary" replace="true">

				      <tree>
				         <!--
				            Whether the folder Tree component should enumerate child folders or not.
				            This is a relatively expensive operation, so should be set to "false" for Repositories with broad folder structures.
				         -->
				         <evaluate-child-folders>false</evaluate-child-folders>
				         
				         <!--
				            Optionally limit the number of folders shown in treeview throughout Share.
				         -->
				         <maximum-folder-count>1000</maximum-folder-count>
				         
				         <!--  
				            Default timeout in milliseconds for folder Tree component to recieve response from Repository
				         -->
				         <timeout>7000</timeout>
				      </tree>

				      <!--
				         Used by the "Manage Aspects" action

				         For custom aspects, remember to also add the relevant i18n string(s)
				            cm_myaspect=My Aspect
				      -->
				      <aspects>
				         <!-- Aspects that a user can see -->
				         <visible>
				            <aspect name="cm:generalclassifiable" />
				            <aspect name="cm:complianceable" />
				            <aspect name="cm:dublincore" />
				            <aspect name="cm:effectivity" />
				            <aspect name="cm:summarizable" />
				            <aspect name="cm:versionable" />
				            <aspect name="cm:templatable" />
				            <aspect name="cm:emailed" />
				            <aspect name="emailserver:aliasable" />
				            <aspect name="cm:taggable" />
				            <aspect name="app:inlineeditable" />
				            <aspect name="gd:googleEditable" />
				            <aspect name="cm:geographic" />
				            <aspect name="exif:exif" />
				            <aspect name="audio:audio" />
				            <aspect name="cm:indexControl" />
				            <aspect name="dp:restrictable" />
				         </visible>

				         <!-- Aspects that a user can add. Same as "visible" if left empty -->
				         <addable>
				         </addable>

				         <!-- Aspects that a user can remove. Same as "visible" if left empty -->
				         <removeable>
				         </removeable>
				      </aspects>

				      <!--
				         Used by the "Change Type" action

				         Define valid subtypes using the following example:
				            <type name="cm:content">
				               <subtype name="cm:mysubtype" />
				            </type>

				         Remember to also add the relevant i18n string(s):
				            cm_mysubtype=My SubType
				      -->
				      <types>
				         <type name="cm:content">
				         </type>

				         <type name="cm:folder">
				         </type>
				 
				         <type name="trx:transferTarget">
				            <subtype name="trx:fileTransferTarget" />
				         </type>
				      </types>

				      <!--
				         If set, will present a WebDAV link for the current item on the Document and Folder details pages.
				         Also used to generate the "View in Alfresco Explorer" action for folders.
				      -->
				      <repository-url>http://localhost:8080/alfresco</repository-url>

				      <!--
				         Google Docs™ integration
				      -->
				      <google-docs>
				         <!--
				            Enable/disable the Google Docs UI integration (Extra types on Create Content menu, Google Docs actions).
				         -->
				         <enabled>false</enabled>

				         <!--
				            The mimetypes of documents Google Docs allows you to create via the Share interface.
				            The I18N label is created from the "type" attribute, e.g. google-docs.doc=Google Docs&trade; Document
				         -->
				         <creatable-types>
				            <creatable type="doc">application/vnd.openxmlformats-officedocument.wordprocessingml.document</creatable>
				            <creatable type="xls">application/vnd.openxmlformats-officedocument.spreadsheetml.sheet</creatable>
				            <creatable type="ppt">application/vnd.ms-powerpoint</creatable>
				         </creatable-types>
				      </google-docs>

				      <!--
				         File upload configuration
				      -->
				      <file-upload>
				         <!--
				            Adobe Flash™
				            In certain environments, an HTTP request originating from Flash cannot be authenticated using an existing session.
				            See: http://bugs.adobe.com/jira/browse/FP-4830
				            For these cases, it is useful to disable the Flash-based uploader for Share Document Libraries.
				         -->
				         <adobe-flash-enabled>true</adobe-flash-enabled>
				      </file-upload>
				   </config>


				   <!-- Custom DocLibActions config section -->
				   <config evaluator="string-compare" condition="DocLibActions">
				      <actionGroups>
				         <actionGroup id="document-browse">

				            <!-- Simple Repo Actions -->
				            <!--
				            <action index="340" id="document-extract-metadata" />
				            <action index="350" id="document-increment-counter" />
				            -->

				            <!-- Dialog Repo Actions -->
				            <!--
				            <action index="360" id="document-transform" />
				            <action index="370" id="document-transform-image" />
				            <action index="380" id="document-execute-script" />
				            -->

				         </actionGroup>
				      </actionGroups>
				   </config>

				   <!-- Global folder picker config section -->
				   <config evaluator="string-compare" condition="GlobalFolder">
				      <siteTree>
				         <container type="cm:folder">
				            <!-- Use a specific label for this container type in the tree -->
				            <rootLabel>location.path.documents</rootLabel>
				            <!-- Use a specific uri to retreive the child nodes for this container type in the tree -->
				            <uri>slingshot/doclib/treenode/site/{site}/{container}{path}?children={evaluateChildFoldersSite}&amp;max={maximumFolderCountSite}</uri>
				         </container>
				      </siteTree>
				   </config>

				   <!-- Repository Library config section -->
				   <config evaluator="string-compare" condition="RepositoryLibrary" replace="true">
				      <!--
				         Root nodeRef or xpath expression for top-level folder.
				         e.g. alfresco://user/home, /app:company_home/st:sites/cm:site1
				         If using an xpath expression, ensure it is properly ISO9075 encoded here.
				      -->
				      <root-node>alfresco://company/home</root-node>

				      <tree>
				         <!--
				            Whether the folder Tree component should enumerate child folders or not.
				            This is a relatively expensive operation, so should be set to "false" for Repositories with broad folder structures.
				         -->
				         <evaluate-child-folders>false</evaluate-child-folders>
				         
				         <!--
				            Optionally limit the number of folders shown in treeview throughout Share.
				         -->
				         <maximum-folder-count>500</maximum-folder-count>
				      </tree>

				      <!--
				         Whether the link to the Repository Library appears in the header component or not.
				      -->
				      <visible>true</visible>
				   </config>
				   
				   <!-- Kerberos settings -->
				   <!-- To enable kerberos rename this condition to "Kerberos" -->
				   <config evaluator="string-compare" condition="KerberosDisabled" replace="true">
				      <kerberos>
				         <!--
				            Password for HTTP service account.
				            The account name *must* be built from the HTTP server name, in the format :
				               HTTP/<server_name>@<realm>
				            (NB this is because the web browser requests an ST for the
				            HTTP/<server_name> principal in the current realm, so if we're to decode
				            that ST, it has to match.)
				         -->
				         <password>secret</password>
				         <!--
				            Kerberos realm and KDC address.
				         -->
				         <realm>ALFRESCO.ORG</realm>
				         <!--
				            Service Principal Name to use on the repository tier.
				            This must be like: HTTP/host.name@REALM
				         -->
				         <endpoint-spn>HTTP/repository.server.com@ALFRESCO.ORG</endpoint-spn>
				         <!--
				            JAAS login configuration entry name.
				         -->
				         <config-entry>ShareHTTP</config-entry>
				        <!--
				           A Boolean which when true strips the @domain sufix from Kerberos authenticated usernames.
				           Use together with stripUsernameSuffix property in alfresco-global.properties file.
				        -->
				        <stripUserNameSuffix>true</stripUserNameSuffix>
				      </kerberos>
				   </config>

				   <!-- Uncomment and modify the URL to Activiti Admin Console if required. -->
				   <!--
				   <config evaluator="string-compare" condition="ActivitiAdmin" replace="true">
				      <activiti-admin-url>http://localhost:8080/alfresco/activiti-admin</activiti-admin-url>
				   </config>
				   -->

				   <config evaluator="string-compare" condition="Remote">
				      <remote>
				         <endpoint>
				            <id>alfresco-noauth</id>
				            <name>Alfresco - unauthenticated access</name>
				            <description>Access to Alfresco Repository WebScripts that do not require authentication</description>
				            <connector-id>alfresco</connector-id>
				            <endpoint-url>http://localhost:8080/alfresco/s</endpoint-url>
				            <identity>none</identity>
				         </endpoint>

				         <endpoint>
				            <id>alfresco</id>
				            <name>Alfresco - user access</name>
				            <description>Access to Alfresco Repository WebScripts that require user authentication</description>
				            <connector-id>alfresco</connector-id>
				            <endpoint-url>http://localhost:8080/alfresco/s</endpoint-url>
				            <identity>user</identity>
				         </endpoint>

				         <endpoint>
				            <id>alfresco-feed</id>
				            <name>Alfresco Feed</name>
				            <description>Alfresco Feed - supports basic HTTP authentication via the EndPointProxyServlet</description>
				            <connector-id>http</connector-id>
				            <endpoint-url>http://localhost:8080/alfresco/s</endpoint-url>
				            <basic-auth>true</basic-auth>
				            <identity>user</identity>
				         </endpoint>
				         <!-- 
				         <endpoint>
				            <id>activiti-admin</id>
				            <name>Activiti Admin UI - user access</name>
				            <description>Access to Activiti Admin UI, that requires user authentication</description>
				            <connector-id>activiti-admin-connector</connector-id>
				            <endpoint-url>http://localhost:8080/alfresco/activiti-admin</endpoint-url>
				            <identity>user</identity>
				         </endpoint>
				         -->
				      </remote>
				   </config>

				   <!-- 
				        Overriding endpoints to reference an Alfresco server with external SSO enabled
				        NOTE: If utilising a load balancer between web-tier and repository cluster, the "sticky
				              sessions" feature of your load balancer must be used.
				        NOTE: If alfresco server location is not localhost:8080 then also combine changes from the
				              "example port config" section below.
				        *Optional* keystore contains SSL client certificate + trusted CAs.
				        Used to authenticate share to an external SSO system such as CAS
				        Remove the keystore section if not required i.e. for NTLM.
				        
				        NOTE: For Kerberos SSO rename the "KerberosDisabled" condition above to "Kerberos"
				        
				        NOTE: For external SSO, switch the endpoint connector to "AlfrescoHeader" and set
				              the userHeader to the name of the HTTP header that the external SSO
				              uses to provide the authenticated user name.
				   -->
				  
				   <config evaluator="string-compare" condition="Remote">
				      <remote>
				         <keystore>
				             <path>alfresco/web-extension/alfresco-system.p12</path>
				             <type>pkcs12</type>
				             <password>alfresco-system</password>
				         </keystore>
				         
				         <connector>
				            <id>alfrescoCookie</id>
				            <name>Alfresco Connector</name>
				            <description>Connects to an Alfresco instance using cookie-based authentication</description>
				            <class>org.alfresco.web.site.servlet.SlingshotAlfrescoConnector</class>
				         </connector>
				      <!--   
				         <connector>
				            <id>alfrescoHeader</id>
				            <name>Alfresco Connector</name>
				            <description>Connects to an Alfresco instance using header and cookie-based authentication</description>
				            <class>org.alfresco.web.site.servlet.SlingshotAlfrescoConnector</class>
				            <userHeader>SsoUserHeader</userHeader>
				         </connector>
				     -->
				         <endpoint>
				            <id>alfresco</id>
				            <name>Alfresco - user access</name>
				            <description>Access to Alfresco Repository WebScripts that require user authentication</description>
				            <connector-id>alfrescoCookie</connector-id>
				            <endpoint-url>http://localhost:8080/alfresco/wcs</endpoint-url>
				            <identity>user</identity>
				            <external-auth>true</external-auth>
				         </endpoint>
				      </remote>
				   </config>
				  
				   <!-- Cookie settings -->
				   <!-- To disable alfUsername2 cookie set enableCookie value to "false" -->
				   <!--
				   <plug-ins>
				      <element-readers>
				         <element-reader element-name="cookie" class="org.alfresco.web.config.cookie.CookieElementReader"/>
				      </element-readers>
				   </plug-ins>
				   
				   <config evaluator="string-compare" condition="Cookie" replace="true">
				      <cookie>
				         <enableCookie>false</enableCookie>
				         
				         <cookies-to-remove>
				            <cookie-to-remove>alfUsername3</cookie-to-remove>
				            <cookie-to-remove>alfLogin</cookie-to-remove>
				         </cookies-to-remove>
				      </cookie>
				   </config>
				   -->
				</alfresco-config>

		- Debugging

			log4j.logger.org.alfresco.passthru.auth=debug (may be more correct than the next ...)
			log4j.logger.org.alfresco.repo.security.authentication.passthru=debug

		- Troubleshooting / Known Issues

			https://issues.alfresco.com/jira/issues/?jql=affectedVersion%20in%20(4.1.0.0%2C%204.1.1%2C%204.1.10%2C%204.1.2%2C%204.1.3%2C%204.1.4%2C%204.1.5%2C%204.1.6%2C%204.1.7%2C%204.1.8%2C%204.1.9%2C%204.2.0%2C%204.2.1%2C%204.2.2%2C%204.2.3%2C%204.2.4%2C%204.2.5%2C%204.2.6%2C%205.0.0%2C%205.0.1%2C%205.0.2%2C%205.0.3%2C%205.0.4%2C%205.1.0%2C%205.1.1%2C%205.1.2)%20AND%20text%20~%20%22kerberos%22

	
	Kerberos

		- Documentation

			http://docs.alfresco.com/5.1/concepts/auth-kerberos-intro.html (intro)
			https://ssimo.org/blog/id_019.html (load-balancers and Kerberos)
			http://docs.alfresco.com/5.1/concepts/auth-kerberos-props.html (kerberos properties)
			http://docs.alfresco.com/5.1/tasks/auth-kerberos-ADconfig.html (kerberos and AD)
			http://docs.alfresco.com/5.1/concepts/auth-kerberos-clientconfig.html (kerberos and client settings)
			http://docs.alfresco.com/5.1/tasks/auth-kerberos-cross-domain.html (multi domains)
			http://docs.alfresco.com/5.1/tasks/auth-kerberos-shareSSO.html (Share SSO)

		- Supported Platforms and Notes

			N/A

		- Downloads (if applicable)

			N/A

		- Setup / Recipes

			Global Properties

				### User synchronization ###
				ldap.authentication.active=false
				ldap.authentication.allowGuestLogin=true
				ldap.authentication.userNameFormat=%s@example.foo
				ldap.authentication.java.naming.factory.initial=com.sun.jndi.ldap.LdapCtxFactory
				ldap.authentication.java.naming.provider.url=ldap://192.168.56.101:389
				ldap.authentication.java.naming.security.authentication=simple
				ldap.authentication.escapeCommasInBind=false
				ldap.authentication.escapeCommasInUid=false
				ldap.authentication.defaultAdministratorUserNames=Administrator

				ldap.synchronization.active=true
				ldap.synchronization.java.naming.security.principal=administrator@example.foo
				ldap.synchronization.java.naming.security.credentials=Alfr3sc0
				ldap.synchronization.queryBatchSize=1000
				ldap.synchronization.attributeBatchSize=1000
				synchronization.synchronizeChangesOnly=false
				synchronization.allowDeletions=true
				synchronization.syncWhenMissingPeopleLogIn=true

				ldap.synchronization.groupQuery=objectclass\=group
				ldap.synchronization.groupDifferentialQuery=(&(objectclass\=group)(!(modifyTimestamp<\={0})))

				ldap.synchronization.personQuery=(&(objectclass\=user)(userAccountControl\:1.2.840.113556.1.4.803\:\=512)(|(memberOf=cn\=AlfrescoAdmins,ou=alfresco,dc=example,dc=foo)(memberOf=cn\=AlfrescoUsers,ou=alfresco,dc=example,dc=foo)))

				ldap.synchronization.personDifferentialQuery=(&(objectclass\=user)(userAccountControl\:1.2.840.113556.1.4.803\:\=512)(|(memberOf=cn\=AlfrescoAdmins,ou=alfresco,dc=example,dc=foo)(memberOf=cn\=AlfrescoUsers,ou=alfresco,dc=example,dc=foo))(!(modifyTimestamp<\={0})))

				ldap.synchronization.groupSearchBase=ou\=alfresco,dc\=example,dc\=foo

				ldap.synchronization.userSearchBase=dc\=example,dc\=foo

				ldap.synchronization.modifyTimestampAttributeName=modifyTimestamp
				ldap.synchronization.timestampFormat=yyyyMMddHHmmss'.0Z'
				ldap.synchronization.userIdAttributeName=sAMAccountName
				ldap.synchronization.userFirstNameAttributeName=givenName
				ldap.synchronization.userLastNameAttributeName=sn
				ldap.synchronization.userEmailAttributeName=mail
				ldap.synchronization.userOrganizationalIdAttributeName=company
				ldap.synchronization.defaultHomeFolderProvider=largeHomeFolderProvider
				ldap.synchronization.groupIdAttributeName=cn
				ldap.synchronization.groupDisplayNameAttributeName=displayName
				ldap.synchronization.groupType=group
				ldap.synchronization.personType=user
				ldap.synchronization.groupMemberAttributeName=member
				ldap.synchronization.enableProgressEstimation=true


				### Kerberos settings ###
				authentication.chain=kerberos1:kerberos,alfrescoNtlm1:alfrescoNtlm
				kerberos.authentication.realm=EXAMPLE.FOO
				kerberos.authentication.authenticateCIFS=true
				kerberos.authentication.cifs.password=Alfr3sc0
				kerberos.authentication.http.password=Alfr3sc0
				kerberos.authentication.sso.enabled=true
				kerberos.authentication.defaultAdministratorUserNames=administrator

			Share Config

				<alfresco-config>

			   <!-- Global config section -->
			   <config replace="true">
			      <flags>
			         <!--
			            Developer debugging setting to turn on DEBUG mode for client scripts in the browser
			         -->
			         <client-debug>false</client-debug>

			         <!--
			            LOGGING can always be toggled at runtime when in DEBUG mode (Ctrl, Ctrl, Shift, Shift).
			            This flag automatically activates logging on page load.
			         -->
			         <client-debug-autologging>false</client-debug-autologging>
			      </flags>
			   </config>
			   
			   <config evaluator="string-compare" condition="WebFramework">
			      <web-framework>
			         <!-- SpringSurf Autowire Runtime Settings -->
			         <!-- 
			              Developers can set mode to 'development' to disable; SpringSurf caches,
			              FreeMarker template caching and Rhino JavaScript compilation.
			         -->
			         <autowire>
			            <!-- Pick the mode: "production" or "development" -->
			            <mode>production</mode>
			         </autowire>

			         <!-- Allows extension modules with <auto-deploy> set to true to be automatically deployed -->
			         <module-deployment>
			            <mode>manual</mode>
			            <enable-auto-deploy-modules>true</enable-auto-deploy-modules>
			         </module-deployment>
			      </web-framework>
			   </config>

			   <!-- Disable the CSRF Token Filter -->
			   <!--
			   <config evaluator="string-compare" condition="CSRFPolicy" replace="true">
			      <filter/>
			   </config>
			   -->

			   <!--
			      To run the CSRF Token Filter behind 1 or more proxies that do not rewrite the Origin or Referere headers:

			      1. Copy the "CSRFPolicy" default config in share-security-config.xml and paste it into this file.
			      2. Replace the old config by setting the <config> element's "replace" attribute to "true" like below:
			         <config evaluator="string-compare" condition="CSRFPolicy" replace="true">
			      3. To every <action name="assertReferer"> element add the following child element
			         <param name="referer">http://www.proxy1.com/.*|http://www.proxy2.com/.*</param>
			      4. To every <action name="assertOrigin"> element add the following child element
			         <param name="origin">http://www.proxy1.com|http://www.proxy2.com</param>
			   -->

			   <!--
			      Remove the default wildcard setting and use instead a strict whitelist of the only domains that shall be allowed
			      to be used inside iframes (i.e. in the WebView dashlet on the dashboards)
			   -->
			   <!--
			   <config evaluator="string-compare" condition="IFramePolicy" replace="true">
			      <cross-domain>
			         <url>http://www.trusted-domain-1.com/</url>
			         <url>http://www.trusted-domain-2.com/</url>
			      </cross-domain>
			   </config>
			   -->

			   <!-- Turn off header that stops Share from being displayed in iframes on pages from other domains -->
			   <!--
			   <config evaluator="string-compare" condition="SecurityHeadersPolicy">
			      <headers>
			         <header>
			            <name>X-Frame-Options</name>
			            <enabled>false</enabled>
			         </header>
			      </headers>
			   </config>
			   -->

			   <!-- Prevent browser communication over HTTP (for HTTPS servers) -->
			   <!--
			   <config evaluator="string-compare" condition="SecurityHeadersPolicy">
			      <headers>
			         <header>
			            <name>Strict-Transport-Security</name>
			            <value>max-age=31536000</value>
			         </header>
			      </headers>
			   </config>
			   -->

			   <config evaluator="string-compare" condition="Replication">
			      <share-urls>
			         <!--
			            To discover a Repository Id, browse to the remote server's CMIS landing page at:
			              http://{server}:{port}/alfresco/service/cmis/index.html
			            The Repository Id field is found under the "CMIS Repository Information" expandable panel.

			            Example config entry:
			              <share-url repositoryId="622f9533-2a1e-48fe-af4e-ee9e41667ea4">http://new-york-office:8080/share/</share-url>
			         -->
			      </share-urls>
			   </config>

			   <!-- Document Library config section -->
			   <config evaluator="string-compare" condition="DocumentLibrary" replace="true">

			      <tree>
			         <!--
			            Whether the folder Tree component should enumerate child folders or not.
			            This is a relatively expensive operation, so should be set to "false" for Repositories with broad folder structures.
			         -->
			         <evaluate-child-folders>false</evaluate-child-folders>
			         
			         <!--
			            Optionally limit the number of folders shown in treeview throughout Share.
			         -->
			         <maximum-folder-count>1000</maximum-folder-count>
			         
			         <!--  
			            Default timeout in milliseconds for folder Tree component to recieve response from Repository
			         -->
			         <timeout>7000</timeout>
			      </tree>

			      <!--
			         Used by the "Manage Aspects" action

			         For custom aspects, remember to also add the relevant i18n string(s)
			            cm_myaspect=My Aspect
			      -->
			      <aspects>
			         <!-- Aspects that a user can see -->
			         <visible>
			            <aspect name="cm:generalclassifiable" />
			            <aspect name="cm:complianceable" />
			            <aspect name="cm:dublincore" />
			            <aspect name="cm:effectivity" />
			            <aspect name="cm:summarizable" />
			            <aspect name="cm:versionable" />
			            <aspect name="cm:templatable" />
			            <aspect name="cm:emailed" />
			            <aspect name="emailserver:aliasable" />
			            <aspect name="cm:taggable" />
			            <aspect name="app:inlineeditable" />
			            <aspect name="gd:googleEditable" />
			            <aspect name="cm:geographic" />
			            <aspect name="exif:exif" />
			            <aspect name="audio:audio" />
			            <aspect name="cm:indexControl" />
			            <aspect name="dp:restrictable" />
			         </visible>

			         <!-- Aspects that a user can add. Same as "visible" if left empty -->
			         <addable>
			         </addable>

			         <!-- Aspects that a user can remove. Same as "visible" if left empty -->
			         <removeable>
			         </removeable>
			      </aspects>

			      <!--
			         Used by the "Change Type" action

			         Define valid subtypes using the following example:
			            <type name="cm:content">
			               <subtype name="cm:mysubtype" />
			            </type>

			         Remember to also add the relevant i18n string(s):
			            cm_mysubtype=My SubType
			      -->
			      <types>
			         <type name="cm:content">
			         </type>

			         <type name="cm:folder">
			         </type>
			 
			         <type name="trx:transferTarget">
			            <subtype name="trx:fileTransferTarget" />
			         </type>
			      </types>

			      <!--
			         If set, will present a WebDAV link for the current item on the Document and Folder details pages.
			         Also used to generate the "View in Alfresco Explorer" action for folders.
			      -->
			      <repository-url>http://localhost:8080/alfresco</repository-url>

			      <!--
			         Google Docs™ integration
			      -->
			      <google-docs>
			         <!--
			            Enable/disable the Google Docs UI integration (Extra types on Create Content menu, Google Docs actions).
			         -->
			         <enabled>false</enabled>

			         <!--
			            The mimetypes of documents Google Docs allows you to create via the Share interface.
			            The I18N label is created from the "type" attribute, e.g. google-docs.doc=Google Docs&trade; Document
			         -->
			         <creatable-types>
			            <creatable type="doc">application/vnd.openxmlformats-officedocument.wordprocessingml.document</creatable>
			            <creatable type="xls">application/vnd.openxmlformats-officedocument.spreadsheetml.sheet</creatable>
			            <creatable type="ppt">application/vnd.ms-powerpoint</creatable>
			         </creatable-types>
			      </google-docs>

			      <!--
			         File upload configuration
			      -->
			      <file-upload>
			         <!--
			            Adobe Flash™
			            In certain environments, an HTTP request originating from Flash cannot be authenticated using an existing session.
			            See: http://bugs.adobe.com/jira/browse/FP-4830
			            For these cases, it is useful to disable the Flash-based uploader for Share Document Libraries.
			         -->
			         <adobe-flash-enabled>true</adobe-flash-enabled>
			      </file-upload>
			   </config>


			   <!-- Custom DocLibActions config section -->
			   <config evaluator="string-compare" condition="DocLibActions">
			      <actionGroups>
			         <actionGroup id="document-browse">

			            <!-- Simple Repo Actions -->
			            <!--
			            <action index="340" id="document-extract-metadata" />
			            <action index="350" id="document-increment-counter" />
			            -->

			            <!-- Dialog Repo Actions -->
			            <!--
			            <action index="360" id="document-transform" />
			            <action index="370" id="document-transform-image" />
			            <action index="380" id="document-execute-script" />
			            -->

			         </actionGroup>
			      </actionGroups>
			   </config>

			   <!-- Global folder picker config section -->
			   <config evaluator="string-compare" condition="GlobalFolder">
			      <siteTree>
			         <container type="cm:folder">
			            <!-- Use a specific label for this container type in the tree -->
			            <rootLabel>location.path.documents</rootLabel>
			            <!-- Use a specific uri to retreive the child nodes for this container type in the tree -->
			            <uri>slingshot/doclib/treenode/site/{site}/{container}{path}?children={evaluateChildFoldersSite}&amp;max={maximumFolderCountSite}</uri>
			         </container>
			      </siteTree>
			   </config>

			   <!-- Repository Library config section -->
			   <config evaluator="string-compare" condition="RepositoryLibrary" replace="true">
			      <!--
			         Root nodeRef or xpath expression for top-level folder.
			         e.g. alfresco://user/home, /app:company_home/st:sites/cm:site1
			         If using an xpath expression, ensure it is properly ISO9075 encoded here.
			      -->
			      <root-node>alfresco://company/home</root-node>

			      <tree>
			         <!--
			            Whether the folder Tree component should enumerate child folders or not.
			            This is a relatively expensive operation, so should be set to "false" for Repositories with broad folder structures.
			         -->
			         <evaluate-child-folders>false</evaluate-child-folders>
			         
			         <!--
			            Optionally limit the number of folders shown in treeview throughout Share.
			         -->
			         <maximum-folder-count>500</maximum-folder-count>
			      </tree>

			      <!--
			         Whether the link to the Repository Library appears in the header component or not.
			      -->
			      <visible>true</visible>
			   </config>
			   
			   <!-- Kerberos settings -->
			   <!-- To enable kerberos rename this condition to "Kerberos" -->
			   <config evaluator="string-compare" condition="Kerberos" replace="true">
			      <kerberos>
			         <!--
			            Password for HTTP service account.
			            The account name *must* be built from the HTTP server name, in the format :
			               HTTP/<server_name>@<realm>
			            (NB this is because the web browser requests an ST for the
			            HTTP/<server_name> principal in the current realm, so if we're to decode
			            that ST, it has to match.)
			         -->
			         <password>Alfr3sc0</password>
			         <!--
			            Kerberos realm and KDC address.
			         -->
			         <realm>EXAMPLE.FOO</realm>
			         <!--
			            Service Principal Name to use on the repository tier.
			            This must be like: HTTP/host.name@REALM
			         -->
			         <endpoint-spn>HTTP/alfrescodemo.com@EXAMPLE.FOO</endpoint-spn>
			         <!--
			            JAAS login configuration entry name.
			         -->
			         <config-entry>ShareHTTP</config-entry>
			        <!--
			           A Boolean which when true strips the @domain sufix from Kerberos authenticated usernames.
			           Use together with stripUsernameSuffix property in alfresco-global.properties file.
			        -->
			        <stripUserNameSuffix>true</stripUserNameSuffix>
			      </kerberos>
			   </config>

			   <!-- Uncomment and modify the URL to Activiti Admin Console if required. -->
			   <!--
			   <config evaluator="string-compare" condition="ActivitiAdmin" replace="true">
			      <activiti-admin-url>http://localhost:8080/alfresco/activiti-admin</activiti-admin-url>
			   </config>
			   -->

			   <config evaluator="string-compare" condition="Remote">
			      <remote>
			         <endpoint>
			            <id>alfresco-noauth</id>
			            <name>Alfresco - unauthenticated access</name>
			            <description>Access to Alfresco Repository WebScripts that do not require authentication</description>
			            <connector-id>alfresco</connector-id>
			            <endpoint-url>http://localhost:8080/alfresco/s</endpoint-url>
			            <identity>none</identity>
			         </endpoint>

			         <endpoint>
			            <id>alfresco</id>
			            <name>Alfresco - user access</name>
			            <description>Access to Alfresco Repository WebScripts that require user authentication</description>
			            <connector-id>alfresco</connector-id>
			            <endpoint-url>http://localhost:8080/alfresco/s</endpoint-url>
			            <identity>user</identity>
			         </endpoint>

			         <endpoint>
			            <id>alfresco-feed</id>
			            <name>Alfresco Feed</name>
			            <description>Alfresco Feed - supports basic HTTP authentication via the EndPointProxyServlet</description>
			            <connector-id>http</connector-id>
			            <endpoint-url>http://localhost:8080/alfresco/s</endpoint-url>
			            <basic-auth>true</basic-auth>
			            <identity>user</identity>
			         </endpoint>
			         <!-- 
			         <endpoint>
			            <id>activiti-admin</id>
			            <name>Activiti Admin UI - user access</name>
			            <description>Access to Activiti Admin UI, that requires user authentication</description>
			            <connector-id>activiti-admin-connector</connector-id>
			            <endpoint-url>http://localhost:8080/alfresco/activiti-admin</endpoint-url>
			            <identity>user</identity>
			         </endpoint>
			         -->
			      </remote>
			   </config>

			   <!-- 
			        Overriding endpoints to reference an Alfresco server with external SSO enabled
			        NOTE: If utilising a load balancer between web-tier and repository cluster, the "sticky
			              sessions" feature of your load balancer must be used.
			        NOTE: If alfresco server location is not localhost:8080 then also combine changes from the
			              "example port config" section below.
			        *Optional* keystore contains SSL client certificate + trusted CAs.
			        Used to authenticate share to an external SSO system such as CAS
			        Remove the keystore section if not required i.e. for NTLM.
			        
			        NOTE: For Kerberos SSO rename the "KerberosDisabled" condition above to "Kerberos"
			        
			        NOTE: For external SSO, switch the endpoint connector to "AlfrescoHeader" and set
			              the userHeader to the name of the HTTP header that the external SSO
			              uses to provide the authenticated user name.
			   -->
			  
			   <config evaluator="string-compare" condition="Remote">
			      <remote>
			         <keystore>
			             <path>alfresco/web-extension/alfresco-system.p12</path>
			             <type>pkcs12</type>
			             <password>alfresco-system</password>
			         </keystore>
			         
			         <connector>
			            <id>alfrescoCookie</id>
			            <name>Alfresco Connector</name>
			            <description>Connects to an Alfresco instance using cookie-based authentication</description>
			            <class>org.alfresco.web.site.servlet.SlingshotAlfrescoConnector</class>
			         </connector>
			      <!--   
			         <connector>
			            <id>alfrescoHeader</id>
			            <name>Alfresco Connector</name>
			            <description>Connects to an Alfresco instance using header and cookie-based authentication</description>
			            <class>org.alfresco.web.site.servlet.SlingshotAlfrescoConnector</class>
			            <userHeader>SsoUserHeader</userHeader>
			         </connector>
			     -->
			         <endpoint>
			            <id>alfresco</id>
			            <name>Alfresco - user access</name>
			            <description>Access to Alfresco Repository WebScripts that require user authentication</description>
			            <connector-id>alfrescoCookie</connector-id>
			            <endpoint-url>http://localhost:8080/alfresco/wcs</endpoint-url>
			            <identity>user</identity>
			            <external-auth>true</external-auth>
			         </endpoint>
			      </remote>
			   </config>
			  
			   <!-- Cookie settings -->
			   <!-- To disable alfUsername2 cookie set enableCookie value to "false" -->
			   <!--
			   <plug-ins>
			      <element-readers>
			         <element-reader element-name="cookie" class="org.alfresco.web.config.cookie.CookieElementReader"/>
			      </element-readers>
			   </plug-ins>
			   
			   <config evaluator="string-compare" condition="Cookie" replace="true">
			      <cookie>
			         <enableCookie>false</enableCookie>
			         
			         <cookies-to-remove>
			            <cookie-to-remove>alfUsername3</cookie-to-remove>
			            <cookie-to-remove>alfLogin</cookie-to-remove>
			         </cookies-to-remove>
			      </cookie>
			   </config>
			   -->
			</alfresco-config>


		- Debugging

			log4j.logger.org.alfresco.web.app.servlet.KerberosAuthenticationFilter=debug
			log4j.logger.org.alfresco.repo.webdav.auth.KerberosAuthenticationFilter=debug

		- Troubleshooting / Known Issues

			https://issues.alfresco.com/jira/issues/?jql=affectedVersion%20in%20(4.1.0.0%2C%204.1.1%2C%204.1.10%2C%204.1.2%2C%204.1.3%2C%204.1.4%2C%204.1.5%2C%204.1.6%2C%204.1.7%2C%204.1.8%2C%204.1.9%2C%204.2.0%2C%204.2.1%2C%204.2.2%2C%204.2.3%2C%204.2.4%2C%204.2.5%2C%204.2.6%2C%205.0.0%2C%205.0.1%2C%205.0.2%2C%205.0.3%2C%205.0.4%2C%205.1.0%2C%205.1.1%2C%205.1.2)%20AND%20text%20~%20%22kerberos%22

	
	External 

		- Documentation

			http://docs.alfresco.com/5.1/concepts/auth-external-intro.html (intro)
			http://docs.alfresco.com/5.1/concepts/auth-basics.html (sso, cas)
			http://docs.alfresco.com/5.1/concepts/auth-external-props.html (properties)
			http://docs.alfresco.com/5.1/tasks/auth-alfrescoexternal-sso.html (with Share sso)
			http://docs.alfresco.com/5.1/tasks/alf-sso-client-certificate.html (client cert)

		- Supported Platforms and Notes

			N/A

		- Downloads (if applicable)

			N/A

		- Setup / Recipes

			N/A

		- Debugging

			Alfresco

				log4j.logger.org.alfresco.web.site.servlet.SSOAuthenticationFilter=debug
			    log4j.logger.org.alfresco.repo.security.authentication.AuthenticationUtil=debug
			    log4j.logger.org.alfresco.repo.security.authentication.AbstractChainingAuthenticationService=debug

			    log4j.logger.org.alfresco.repo.security.authentication=debug

			Share

				log4j.logger.org.alfresco.web.app.servlet.DefaultRemoteUserMap=debug
			    log4j.logger.org.springframework.extensions.webscripts.connector.RemoteClient=debug
			    log4j.logger.org.springframework.extensions.webscripts.connector.AlfrescoAuthenticator=debug


		- Troubleshooting / Known Issues

			https://issues.alfresco.com/jira/issues/?jql=affectedVersion%20in%20(4.1.0.0%2C%204.1.1%2C%204.1.10%2C%204.1.2%2C%204.1.3%2C%204.1.4%2C%204.1.5%2C%204.1.6%2C%204.1.7%2C%204.1.8%2C%204.1.9%2C%204.2.0%2C%204.2.1%2C%204.2.2%2C%204.2.3%2C%204.2.4%2C%204.2.5%2C%204.2.6%2C%205.0.0%2C%205.0.1%2C%205.0.2%2C%205.0.3%2C%205.0.4%2C%205.1.0%2C%205.1.1%2C%205.1.2)%20AND%20text%20~%20%22external%20auth%22

	
	Cloud SAML

		- Documentation

			http://docs.alfresco.com/cloud/concepts/SAML_overview.html (overview)
			http://docs.alfresco.com/cloud/tasks/config_saml.html (configs)
			http://docs.alfresco.com/cloud/tasks/configuring_identityprovider_SAML.html (Ping Federate)
			http://docs.alfresco.com/cloud/tasks/saml-test.html (testing)
			http://docs.alfresco.com/cloud/concepts/SAML-troubleshooting.html

		- Supported Platforms and Notes

			Cloud only as of 2/19/2017

		- Downloads (if applicable)

			N/A

		- Setup / Recipes

			N/A

		- Debugging

			N/A

		- Troubleshooting / Known Issues

			See troubleshooting above

			https://issues.alfresco.com/jira/issues/?jql=affectedVersion%20in%20(%22Cloud%2031%22%2C%20%22Cloud%2032%22%2C%20%22Cloud%2033%22%2C%20%22Cloud%2034%22%2C%20%22Cloud%2034.1%22%2C%20%22Cloud%2035%22%2C%20%22Cloud%2035.1%22%2C%20%22Cloud%2036%22%2C%20%22Cloud%2037%22%2C%20%22Cloud%2038%22%2C%20%22Cloud%2038.2%22%2C%20%22Cloud%2038.3%22%2C%20%22Cloud%2039%22%2C%20%22Cloud%2039.1%22%2C%20%22Cloud%2039.2%22%2C%20%22Cloud%2039.3%22%2C%20%22Cloud%2039.4%22%2C%20%22Cloud%2039.5%22%2C%20%22Cloud%2039.6%22%2C%20%22Cloud%2039.6.1%22%2C%20%22Cloud%2039.6.2%22%2C%20%22Cloud%2039.6.3%22%2C%20%22Cloud%2039.6.4%22%2C%20%22Cloud%2039.6.5%22%2C%20%22Cloud%2039.6.6%22%2C%20%22Cloud%2040%22%2C%20%22Cloud%2041%22%2C%20%22Cloud%2042%22%2C%20%22Cloud%2042.1%22%2C%20%22Cloud%2042.2%22%2C%20%22Cloud%2042.3%22%2C%20%22Cloud%2043%22%2C%20%22Cloud%2043.1%22%2C%20%22Cloud%2044%22%2C%20%22Cloud%2044.2%22%2C%20%22Cloud%2044.3%22%2C%20%22Cloud%2045%22)%20AND%20text%20~%20%22saml%22


	On Premise SAML (unsupported for now)

		- Documentation

		- Supported Platforms and Notes

		- Downloads (if applicable)

		- Setup / Recipes

		- Debugging

		- Troubleshooting / Known Issues

			https://issues.alfresco.com/jira/issues/?jql=affectedVersion%20in%20(4.1.0.0%2C%204.1.1%2C%204.1.10%2C%204.1.2%2C%204.1.3%2C%204.1.4%2C%204.1.5%2C%204.1.6%2C%204.1.7%2C%204.1.8%2C%204.1.9%2C%204.2.0%2C%204.2.1%2C%204.2.2%2C%204.2.3%2C%204.2.4%2C%204.2.5%2C%204.2.6%2C%205.0.0%2C%205.0.1%2C%205.0.2%2C%205.0.3%2C%205.0.4%2C%205.1.0%2C%205.1.1%2C%205.1.2)%20AND%20text%20~%20%22saml%22


Box Connector (Can't find much on this - clarify)


CIFS

	General

		- Documentation

			http://docs.alfresco.com/5.1/concepts/fileserv-CIFS-props.html (properties)

		- Debug

			To help you diagnose CIFS issues (along with using wireshark, jstacks and threaddumps) you can increase Alfresco application logging for CIFS. To do so you will need to add additional CIFS server debug flags to your Alfresco configuration.  After adding the additional CIFS flags, toggle on/off the additional logging setting two log4j properties to debug (i.e. fileserver, smb.protocol.auth).  While these log4j properties are not set to "debug" mode, you will not see increased CIFS logging occurring in log files. It is advised not to leave the logging on "debug" unless your diagnosing a CIFS issue.

			Notes:

			This information is applicable to any version 3.2 or newer of Alfresco. Dependent on the version, the additional logging has increased as the versions have changed. The 3.4.x and higher versions of Alfresco provide the most application logging for CIFS.

			Option 1

			This does not require restarting of application server.

			Set the additional CIFS flags in jconsole:  "cifs.sessionDebug" in MBean tab:
			Alfresco > Configuration > fileServers > Attributes

			Set optional flags (refer to documentation regarding purpose of each):
			cifs.sessionDebug=NETBIOS,STATE,RXDATA,TXDATA,DUMPDATA,NEGOTIATE,TREE,SEARCH,INFO,FILE,FILEIO,TRANSACT,ECHO,ERROR,IPC,LOCK,PKTTYPE,DCERPC,STATECACHE,TIMING,NOTIFY,STREAMS,SOCKET,PKTPOOL,PKTSTATS,THREADPOOL,BENCHMARK,OPLOCK

			note: It is best to have these flags set permanently. Such that you can toggle them on with debug as needed. They will not log unless you have the fileserver and protocol set to debug (see next step)

			Enable logging on these flags will occur till setting these to debug (do so in jconsole as needed):
			
			org.alfresco.fileserver=debug
			org.alfresco.smb.protocol.auth=debug
			
			note: To stop the additional logging change the fileserver and smb.prototocol.auth settings to infor,warn or error in jconsole.

			Option 2

			note: This process requires a restart to have changes take affect.

			Set the additional CIFS flags in "alfresco-global.properties" file by adding the following:
			cifs.sessionDebug=NETBIOS,STATE,RXDATA,TXDATA,DUMPDATA,NEGOTIATE,TREE,SEARCH,INFO,FILE,FILEIO,TRANSACT,ECHO,ERROR,IPC,LOCK,PKTTYPE,DCERPC,STATECACHE,TIMING,NOTIFY,STREAMS,SOCKET,PKTPOOL,PKTSTATS,THREADPOOL,BENCHMARK,OPLOCK

			Set the following to "debug" in "log4j.properties" file (ex, {tomcat}/webapps/alfresco/WEB-INF/classes/log4j.properties). If these properties are not in the log4j.properties file, add them:
			
			log4j.logger.org.alfresco.fileserver=debug
			log4j.logger.org.alfresco.smb.protocol.auth=debug
			
			Restart application server for changes to take affect. Also requires you to change the logging settings to not be set to "debug" and restart to turn off the additional CIFS logging.
				 
			Related Information	
			http://docs.alfresco.com/5.0/tasks/fileserv-CIFS-adv.html
			https://wiki.alfresco.com/wiki/File_Server_Subsystem_4.0#CIFS_server_debug_flags
			Article #00000196: Alfresco is not allowing any more connections over CIFS
			https://issues.alfresco.com/jira/browse/MNT-7568 (increase CIFS alfresco loggers)

		- Troubleshooting

			https://issues.alfresco.com/jira/issues/?jql=affectedVersion%20in%20(4.1.0.0%2C%204.1.1%2C%204.1.10%2C%204.1.2%2C%204.1.3%2C%204.1.4%2C%204.1.5%2C%204.1.6%2C%204.1.7%2C%204.1.8%2C%204.1.9%2C%204.2.0%2C%204.2.1%2C%204.2.2%2C%204.2.3%2C%204.2.4%2C%204.2.5%2C%204.2.6%2C%205.0.0%2C%205.0.1%2C%205.0.2%2C%205.0.3%2C%205.0.4%2C%205.1.0%2C%205.1.1%2C%205.1.2)%20AND%20text%20~%20%22cifs%22

	Windows

		- Documentation

			http://docs.alfresco.com/5.1/concepts/fileserv-subsystem-info.html (Windows information)

		- Supported Platforms and Notes

			N/A

		- Downloads (if applicable)

			N/A

		- Setup / Recipes

			See this KB article for setup instructions for Windows: https://alfresco.my.salesforce.com/kA9D0000000GmnA?srPos=1&srKp=ka9&lang=en_US

		- Debugging

		- Troubleshooting / Known Issues

			https://issues.alfresco.com/jira/issues/?jql=affectedVersion%20in%20(4.1.0.0%2C%204.1.1%2C%204.1.10%2C%204.1.2%2C%204.1.3%2C%204.1.4%2C%204.1.5%2C%204.1.6%2C%204.1.7%2C%204.1.8%2C%204.1.9%2C%204.2.0%2C%204.2.1%2C%204.2.2%2C%204.2.3%2C%204.2.4%2C%204.2.5%2C%204.2.6%2C%205.0.0%2C%205.0.1%2C%205.0.2%2C%205.0.3%2C%205.0.4%2C%205.1.0%2C%205.1.1%2C%205.1.2)%20AND%20text%20~%20%22cifs%20windows%22

	Linux

		- Documentation

			http://docs.alfresco.com/5.1/concepts/fileserv-CIFS-javaprops.html

				The following properties will only take effect on non-Windows servers, where the Java-based SMB implementation is used.

					cifs.broadcast
					Specifies the broadcast mask for the network.
					
					cifs.bindto
					Specifies the network adapter to which to bind. If not specified, the server will bind to all available adapters/addresses.
					
					cifs.tcpipSMB.port
					Controls the port used to listen for the SMB over TCP/IP protocol (or native SMB), supported by Win2000 and above clients. The default port is 445.
					
					cifs.ipv6.enabled
					Enables the use of IP v6 in addition to IP v4 for native SMB. When true, the server will listen for incoming connections on IPv6 and IPv4 sockets.
					
					cifs.netBIOSSMB.namePort
					Controls the NetBIOS name server port on which to listen. The default is 137.
					
					cifs.netBIOSSMB.datagramPort
					Controls the NetBIOS datagram port. The default is 138.
					
					cifs.netBIOSSMB.sessionPort
					Controls the NetBIOS session port on which to listen for incoming session requests. The default is 139.
					
					cifs.WINS.autoDetectEnabled
					When true causes the cifs.WINS.primary and cifs.WINS.secondary properties to be ignored.
					
					cifs.WINS.primary
					Specifies a primary WINS server with which to register the server name.
					
					cifs.WINS.secondary
					Specifies a secondary WINS server with which to register the server name.
					
					cifs.disableNIO
					Disables the new NIO-based CIFS server code and reverts to using the older socket based code.

			http://docs.alfresco.com/5.1/tasks/fileserv-CIFS-useracc.html (instructions for non-root accounts including firewall info)


		- Supported Platforms and Notes

			N/A

		- Downloads (if applicable)

			N/A

		- Setup / Recipes



		- Debugging

		- Troubleshooting / Known Issues

			https://issues.alfresco.com/jira/issues/?jql=affectedVersion%20in%20(4.1.0.0%2C%204.1.1%2C%204.1.10%2C%204.1.2%2C%204.1.3%2C%204.1.4%2C%204.1.5%2C%204.1.6%2C%204.1.7%2C%204.1.8%2C%204.1.9%2C%204.2.0%2C%204.2.1%2C%204.2.2%2C%204.2.3%2C%204.2.4%2C%204.2.5%2C%204.2.6%2C%205.0.0%2C%205.0.1%2C%205.0.2%2C%205.0.3%2C%205.0.4%2C%205.1.0%2C%205.1.1%2C%205.1.2)%20AND%20text%20~%20%22cifs%20linux%22

	Windows Client

		- Documentation

			http://docs.alfresco.com/5.1/tasks/fileserv-CIFS-adv.html (Windows client info)

		- Supported Platforms and Notes

			N/A

		- Downloads (if applicable)

			N/A

		- Setup / Recipes

			N/A

		- Debugging

			log4j.logger.org.alfresco.smb.protocol=debug
			log4j.logger.org.alfresco.fileserver=debug

		- Troubleshooting / Known Issues

			https://issues.alfresco.com/jira/issues/?jql=affectedVersion%20in%20(4.1.0.0%2C%204.1.1%2C%204.1.10%2C%204.1.2%2C%204.1.3%2C%204.1.4%2C%204.1.5%2C%204.1.6%2C%204.1.7%2C%204.1.8%2C%204.1.9%2C%204.2.0%2C%204.2.1%2C%204.2.2%2C%204.2.3%2C%204.2.4%2C%204.2.5%2C%204.2.6%2C%205.0.0%2C%205.0.1%2C%205.0.2%2C%205.0.3%2C%205.0.4%2C%205.1.0%2C%205.1.1%2C%205.1.2)%20AND%20text%20~%20%22cifs%20windows%20mount%22


	Linux Client

		- Documentation

			Officially not supported but should still work.
			Mac client may be supported however.

		- Supported Platforms and Notes

			See above

		- Downloads (if applicable)

			N/A

		- Setup / Recipes

			mount -t cifs -o username=[admin],password=[admin] //[hostname]/alfresco [/path/to/mount_point]

		- Debugging

			N/A

		- Troubleshooting / Known Issues

			You may get a wrong block type error. You just need to install the cifs-utils package for your OS.

	Mac OS Client

		see Linux client. Should work similarly though it may not be supported.


CMIS/CMIS Extension

	General Debug:

		log4j.logger.org.alfresco.opencmis=trace 
		log4j.logger.org.alfresco.opencmis.AlfrescoCmisServiceInterceptor=trace 
		log4j.logger.org.alfresco.cmis=debug 
		log4j.logger.org.alfresco.cmis.dictionary=debug 
		log4j.logger.org.apache.chemistry.opencmis=debug

		Logging query times for CMIS:

			https://alfresco.my.salesforce.com/kA9D0000000GnSg?srPos=2&srKp=ka9&lang=en_US

	CMIS Workbench

		- Documentation

			http://docs.alfresco.com/5.1/pra/1/concepts/cmis-request-url-format-onpremise.html (atompub url setup)

		- Supported Platforms and Notes

		- Downloads (if applicable)

			http://chemistry.apache.org/java/download.html

		- Setup / Recipes

			Using the workbench, the atompub url is:

				   http://localhost:8080/alfresco/api/-default-/public/cmis/versions/1.1/atom (https:// throws an error)

				   If you want to use 1.0, you can simply change the 1.1 to 1.0

		- Debugging

			N/A

		- Troubleshooting / Known Issues

			https://issues.alfresco.com/jira/issues/?jql=affectedVersion%20in%20(4.1.0.0%2C%204.1.1%2C%204.1.10%2C%204.1.2%2C%204.1.3%2C%204.1.4%2C%204.1.5%2C%204.1.6%2C%204.1.7%2C%204.1.8%2C%204.1.9%2C%204.2.0%2C%204.2.1%2C%204.2.2%2C%204.2.3%2C%204.2.4%2C%204.2.5%2C%204.2.6%2C%205.0.0%2C%205.0.1%2C%205.0.2%2C%205.0.3%2C%205.0.4%2C%205.1.0%2C%205.1.1%2C%205.1.2)%20AND%20text%20~%20%22cmis%20workbench%22

	Java

		- Documentation

			https://chemistry.apache.org/java/opencmis.html (the docs here have changed and there are no longer any code samples)

		- Supported Platforms and Notes

			Should work with any Alfresco version

		- Downloads (if applicable)

			https://chemistry.apache.org/java/download.html

		- Setup / Recipes

			We need to write some.

		- Debugging

			See https://chemistry.apache.org/java/developing/dev-logging.html

		- Troubleshooting / Known Issues

	Python

		- Documentation

			https://chemistry.apache.org/python/docs/

		- Supported Platforms and Notes

		- Downloads (if applicable)

			https://chemistry.apache.org/python/cmislib.html
			You can also just do pip install cmislib for your version of Python.

		- Setup / Recipes

			https://chemistry.apache.org/python/docs/examples.html

		- Debugging

			N/A

		- Troubleshooting / Known Issues

			A *lot* of Java methods have not been implemented.


Desktop Sync

	- Documentation

		http://docs.alfresco.com/desktopsync/concepts/ds-overview.html (overview)
		http://docs.alfresco.com/desktopsync/concepts/desktopsync-settingup.html (one minute video)
		http://docs.alfresco.com/desktopsync/concepts/desktopsync-selectingcontent.html (content - 1.5 minute video)

	- Supported Platforms and Notes

		Windows Client 7
		Also, if you look at the 5.1 supported platforms, there is mention of a 2.1 version but there are no docs specifically for this version (or possibly type -- is this a server/client product?) See:

			Alfresco Desktop Sync Service v2.1 ***

			*** Alfresco Desktop Sync will replicate content on local desktops for users with the appropriate access. If
			replication outside the Alfresco repository is not allowed by your content policy you should not deploy Alfresco
			Desktop Sync.
			This version of Alfresco Desktop Sync does not support Smart Folders and Records Management. If Records
			Management controlled content is synced note that moving, renaming or hiding of declared records may not be
			reflected on the desktop client.”

	- Downloads (if applicable)

		https://releases.alfresco.com/DesktopSync/

	- Setup / Recipes

		See tutorials mentioned in docs section above.

	- Debugging

		See http://docs.alfresco.com/desktopsync/concepts/ds-config.html

	- Troubleshooting / Known Issues

		https://issues.alfresco.com/jira/issues/?jql=affectedVersion%20in%20(4.1.0.0%2C%204.1.1%2C%204.1.10%2C%204.1.2%2C%204.1.3%2C%204.1.4%2C%204.1.5%2C%204.1.6%2C%204.1.7%2C%204.1.8%2C%204.1.9%2C%204.2.0%2C%204.2.1%2C%204.2.2%2C%204.2.3%2C%204.2.4%2C%204.2.5%2C%204.2.6%2C%205.0.0%2C%205.0.1%2C%205.0.2%2C%205.0.3%2C%205.0.4%2C%205.1.0%2C%205.1.1%2C%205.1.2)%20AND%20text%20~%20%22desktop%20sync%22


Event listeners (Can't find much on this - clarify)

Event service (Can't find much on this - clarify)

Evernote connector (Can't find much on this - clarify)


FTP
	- Documentation

		http://docs.alfresco.com/5.1/concepts/fileserv-ftp-intro.html (start)
		http://docs.alfresco.com/5.1/concepts/fileserv-ftp-props.html (ftp properties)
		http://docs.alfresco.com/5.1/tasks/fileserv-ftp-adv.html (advanced spring overrides)

	- Supported Platforms and Notes

		N/A

	- Downloads (if applicable)

		N/A

	- Setup / Recipes

		Set up FTP in AWS environment:
			http://blog.prodigi.us/?p=8

		How do I configure FTPS on Alfresco Enterprise?
			https://alfresco.my.salesforce.com/articles/en_US/Technical_Article/How-do-I-configure-FTPS-on-Alfresco-Enterprise?popup=false&navBack=H4sIAAAAAAAAAIuuVipWslLyzssvz0lNSU_1yM9NVdJRygaKFSSmp4ZkluSA-KVAvn58aaZ-NkyhPpCDosu-ODWxKDnDNq2kQKk2FgCo9HQMWAAAAA

		How to run Alfresco as a non-root user and use it with CIFS and FTP on privileged ports?
			https://alfresco.my.salesforce.com/articles/en_US/Technical_Article/How-to-run-Alfresco-as-a-non-root-user-and-use-it-with-CIFS-and-FTP-on-privileged-ports?popup=false&navBack=H4sIAAAAAAAAAIuuVipWslLyzssvz0lNSU_1yM9NVdJRygaKFSSmp4ZkluSA-KVAvn58aaZ-NkyhPpCDosu-ODWxKDnDNq2kQKk2FgCo9HQMWAAAAA

		FTP Clustering Load Balancing And Performance Questions
			https://alfresco.my.salesforce.com/articles/en_US/Technical_Article/FTP-Clustering-Load-Balancing-And-Performance-Questions?popup=false&navBack=H4sIAAAAAAAAAIuuVipWslLyzssvz0lNSU_1yM9NVdJRygaKFSSmp4ZkluSA-KVAvn58aaZ-NkyhPpCDosu-ODWxKDnDNq2kQKk2FgCo9HQMWAAAAA

		How to turn on FTP session logging
			https://alfresco.my.salesforce.com/articles/en_US/Technical_Article/How-to-turn-on-FTP-session-logging?popup=false&navBack=H4sIAAAAAAAAAIuuVipWslLyzssvz0lNSU_1yM9NVdJRygaKFSSmp4ZkluSA-KVAvn58aaZ-NkyhPpCDosu-ODWxKDnDNq2kQKk2FgCo9HQMWAAAAA

		What is the reason of introducing 'ftp.bindto' property?
			https://alfresco.my.salesforce.com/articles/en_US/Technical_Article/What-is-the-reason-of-introducing-ftp-bindto-property?popup=false&navBack=H4sIAAAAAAAAAIuuVipWslLyzssvz0lNSU_1yM9NVdJRygaKFSSmp4ZkluSA-KVAvn58aaZ-NkyhPpCDosu-ODWxKDnDNq2kQKk2FgCo9HQMWAAAAA

		How do I restart FTP (and potentially any other file server) without restarting Alfresco? (JMX)
			https://alfresco.my.salesforce.com/articles/en_US/Technical_Article/How-do-I-restart-FTP-and-potentially-any-other-file-server-without-restarting-Alfresco?popup=false&navBack=H4sIAAAAAAAAAIuuVipWslLyzssvz0lNSU_1yM9NVdJRygaKFSSmp4ZkluSA-KVAvn58aaZ-NkyhPpCDosu-ODWxKDnDNq2kQKk2FgCo9HQMWAAAAA

		How to make FTP and WebDAV work for OpenLDAP users when Passthru is also configured in the authentication chain.
			https://alfresco.my.salesforce.com/articles/en_US/Technical_Article/How-to-make-FTP-and-WebDAV-work-for-OpenLDAP-users-when-Passthru-is-also-configured-in-the-authentication-chain?popup=false&navBack=H4sIAAAAAAAAAIuuVipWslLyzssvz0lNSU_1yM9NVdJRygaKFSSmp4ZkluSA-KVAvn58aaZ-NkyhPpCDosu-ODWxKDnDNq2kQKk2FgCo9HQMWAAAAA

	- Debugging

		log4j.logger.org.alfresco.ftp.protocol=debug
		log4j.logger.org.alfresco.ftp.server=debug

		Flag		Description
		State		Session state changes
		Search		Folder searches
		Info		File information requests
		File		File open/close
		FileIO		File read/write
		Error		Errors
		Pkttype		Received packet type
		Timing		Time packet processing
		Dataport	Data port
		Directory	Directory commands

	- Troubleshooting / Known Issues

		https://issues.alfresco.com/jira/issues/?jql=affectedVersion%20in%20(4.1.0.0%2C%204.1.1%2C%204.1.10%2C%204.1.2%2C%204.1.3%2C%204.1.4%2C%204.1.5%2C%204.1.6%2C%204.1.7%2C%204.1.8%2C%204.1.9%2C%204.2.0%2C%204.2.1%2C%204.2.2%2C%204.2.3%2C%204.2.4%2C%204.2.5%2C%204.2.6%2C%205.0.0%2C%205.0.1%2C%205.0.2%2C%205.0.3%2C%205.0.4%2C%205.1.0%2C%205.1.1%2C%205.1.2)%20AND%20text%20~%20%22ftp%22

	FTPS

		Look at the options for "ftps" here:
			http://docs.alfresco.com/5.1/concepts/fileserv-ftp-props.html


Google Docs

	- Documentation

		http://docs.alfresco.com/5.1/concepts/googledocs-intro.html
		http://docs.alfresco.com/5.1/tasks/googledocs-amp-install.html (amp install)
		http://docs.alfresco.com/5.1/concepts/googledocs-props.html (specific Google Docs properties)
		http://docs.alfresco.com/5.1/tasks/adminconsole-googledocs.html (see above but this for admin console)

	- Supported Platforms and Notes

		http://docs.alfresco.com/5.1/concepts/googledocs-filetypes.html (supported doc types)

		File type	Description
		DOC			A Microsoft Word 97-2003 document.
		XLS			A Microsoft Excel 97-2003 Workbook.
		PPT			A Microsoft PowerPoint 97-2003 Presentation.
		DOCX		An XML-based Microsoft Word document.
		XLSX		An XML-based Microsoft Excel Workbook.
		PPTX		An XML-based Microsoft PowerPoint presentation.

	- Downloads (if applicable)

		https://releases.alfresco.com/GoogleDocs/

	- Setup / Recipes

		See docs above

	- Debugging

		Alfresco Share Server & Client Side Javascript Debugging Tips

			https://alfresco.my.salesforce.com/articles/en_US/White_Paper/Alfresco-Share-Server-Client-Side-Javascript-Debugging-Tips?popup=false&navBack=H4sIAAAAAAAAAIuuVipWslLyzssvz0lNSU_1yM9NVdJRygaKFSSmp4ZkluSA-KVAvn58aaZ-NkyhPpCDosu-ODWxKDnDNj0_Pz0nVTslP7lYOyU1qTRdqTYWAMBqbZRmAAAA

		log4j.logger.org.alfresco.repo.googledocs=debug


	- Troubleshooting / Known Issues


IMAP Protocol

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues


JLAN 

Kerberos (see auth)


Kofax Connector

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues


License

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues


Media Management

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues


Metadata Extraction

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues


Module Management Tool

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues


Module Framework (AMP?)

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues


NFS (5.1 EOF)

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues


Office Protocols

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues

	SSL


Outlook Plugin

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues


Person 

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues


Rendition

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues


Salesforce

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues


SAML
	
	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues


Sharepoint (See Office Protocols)

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues


SMTP
	
	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues

	SSL


Tenancy

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues


Thumbnail

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues


Transformation

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues


Transformation Server

	- Documentation

		http://docs.alfresco.com/5.1/concepts/transerv-intro.html
		http://docs.alfresco.com/5.1/concepts/transerv-config.html

	- Supported Platforms and Notes

		Office and JDK must be x86

	- Downloads (if applicable)

	- Setup / Recipes

		Installation (for 5.1.x support)

			Install Windows 2012
			
			On Windows 2012, install JDK 8 (Update 5 x86)
			
			Download WinRAR (optional)
			
			Install Office Plus 2013 x86
			
			Verify that Office works as expected.
			
			Install VPN software from w3.alfresco.com (if needed)
			
			Upload Transformation Server (1.5.2) to the Windows 2012 server.
			
			Upload TS to your Alfresco server.
			
			Install the TS on the Windows 2012 Server.
			
			On Windows, in Services, Document Transformation Server should be running.
			
			On the Alfresco server, unzip the TS package and move the amps to amps and 	amps_share folder for 5.1.2 install.

			Add the following to alfresco-global.properties:

				### Transformation Server ###
				transformserver.aliveCheckTimeout=2
				transformserver.test.cronExpression=0/10 * * * * ?
				transformserver.disableSSLCertificateValidation=false
				transformserver.username=alfresco
				transformserver.password=alfresco
				transformserver.qualityPreference=QUALITY
				transformserver.transformationTimeout=300
				transformserver.url=http://tserver1:8080/transformation-server

			Add this debug to log4j.properties:

				log4j.logger.org.alfresco.repo.content.transform.TransformerDebug=DEBUG

			Start Alfresco, upload a document and watch logs for verification.


	- Debugging

		log4j.logger.org.alfresco.repo.content.transform.TransformerDebug=DEBUG

		 Increase logging on the Transform Server it self. This will allow you to see if docs are getting to the server as expected and how they are being transformed.

		Locate in the Transformation Server tomcat install the  "{tomcat}/webapps/transformation-server/WEB-INF/classes/log4j.xml" logging property file and modify logging settings to debug as follows:

			<!-- set to 'debug' to log command lines -->
			<logger name="com.westernacher.wps.alfresco.transformation.shell.ProcessExecuter">
			    <level value="debug" />
			</logger>

			<!-- log command line invocations -->
			<logger name="com.westernacher.wps.alfresco.transformation.transformer.external.CommandLineTransformer">
			    <level value="debug" />
			</logger>

			<logger name="com.westernacher.wps.alfresco.transformation.webscript.service.TransformerWebScript">
			    <level value="debug" />
			</logger>

	- Troubleshooting / Known Issues


Upgrades

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues

	Upgrade path


WebDAV

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues


Workflow

	- Documentation

	- Supported Platforms and Notes

	- Downloads (if applicable)

	- Setup / Recipes

	- Debugging

	- Troubleshooting / Known Issues

