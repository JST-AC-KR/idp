cert-export:
	docker cp iugj_idp:/opt/shibboleth-idp/credentials/idp-encryption.key	~/idp/shibboleth-idp/opt/shibboleth-idp/credentials/
	docker cp iugj_idp:/opt/shibboleth-idp/credentials/idp-encryption.crt	~/idp/shibboleth-idp/opt/shibboleth-idp/credentials/
	docker cp iugj_idp:/opt/shibboleth-idp/credentials/idp-signing.key	~/idp/shibboleth-idp/opt/shibboleth-idp/credentials/
	docker cp iugj_idp:/opt/shibboleth-idp/credentials/idp-signing.crt	~/idp/shibboleth-idp/opt/shibboleth-idp/credentials/
	docker cp iugj_idp:/opt/shibboleth-idp/metadata/idp-metadata.xml	~/idp/shibboleth-idp/opt/shibboleth-idp/credentials/

metadatagen:
	docker exec -it iugj_idp /opt/shibboleth-idp/bin/plugin.sh --noPrompt -I net.shibboleth.idp.plugin.metadatagen
