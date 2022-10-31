cert-export:
	docker cp iugj_idp:/opt/shibboleth-idp/credentials/idp-encryption.key	~/shibboleth-idp/opt/shibboleth-idp/credentials/
	docker cp iugj_idp:/opt/shibboleth-idp/credentials/idp-encryption.crt	~/shibboleth-idp/opt/shibboleth-idp/credentials/
	docker cp iugj_idp:/opt/shibboleth-idp/credentials/idp-signing.key	~/shibboleth-idp/opt/shibboleth-idp/credentials/
	docker cp iugj_idp:/opt/shibboleth-idp/credentials/idp-signing.crt	~/shibboleth-idp/opt/shibboleth-idp/credentials/
	docker cp iugj_idp:/opt/shibboleth-idp/metadata/idp-metadata.xml	~/shibboleth-idp/opt/shibboleth-idp/credentials/

metadatagen:
	docker exec -it iugj_idp /opt/shibboleth-idp/bin/plugin.sh --noPrompt -I net.shibboleth.idp.plugin.metadatagen
