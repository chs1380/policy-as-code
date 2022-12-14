ZIPFILE ?= workshop-stack-app.zip
#S3_ASSET_BUCKET ?= ee-assets-prod-us-east-1
#S3_ASSET_PREFIX ?= modules/7381f5596f1840508211082c797067c0/v2/
S3_ASSET_BUCKET ?= ee-assets-jaknn-sandbox
S3_ASSET_PREFIX ?= modules/local_testing/v1/
S3_URL := s3://$(S3_ASSET_BUCKET)/$(S3_ASSET_PREFIX)$(ZIPFILE)

# Cloud9 memberships won't accept a role ARN - only an assume-role ARN
# Cloud9 owner has to be the actual role ARN - so we need both
TEAM_ROLE_NAME := EventEngineTesting
TEAM_ROLE_ARN := arn:aws:iam::$(ACCOUNT_ID):role/$(TEAM_ROLE_NAME)
TEAM_ROLE_SESSION_NAME := jaknn-Isengard  # This will be '/MasterKey' when running in Event Engine
TEAM_ASSUMED_ROLE_ARN := arn:aws:iam::$(ACCOUNT_ID):assumed-role/$(TEAM_ROLE_NAME)/$(TEAM_ROLE_SESSION_NAME)

.PHONY: upload
upload: $(ZIPFILE)
	@echo Note: If this fails, ensure the proper AWS credentials are set in the environment. These can be found in the Event Engine module configuration.
	aws s3 cp $(ZIPFILE) $(S3_URL)
	aws s3 cp ./scripts/bootstrap.sh s3://$(S3_ASSET_BUCKET)/$(S3_ASSET_PREFIX)
zip: $(ZIPFILE)

$(ACCOUNT_ID): $(shell aws sts get-caller-identity --query 'Account')

$(ZIPFILE): $(shell git ls-files)
	rm -f $@
	cd ../.. && git ls-files | xargs zip cdk/ide/$@

.PHONY: clean
clean:
	rm -f $(ZIPFILE)
	find . -name '*.js' ! -name 'jest.config.js' -not -path './node_modules/*' -delete
	find . -name '*.d.ts' -not -path './node_modules/*' -delete
	rm -rf cdk.out/


.PHONY: build
build:
	npm run build

.PHONY: synth-bootstrap
synth-bootstrap: build
	npx cdk synth FoundationStack

.PHONY: deploy
deploy: build upload
	npx cdk deploy FoundationStack --require-approval=never \
		--previous-parameters=false \
		--parameters FoundationStack:EEAssetsBucket=$(S3_ASSET_BUCKET) \
		--parameters FoundationStack:EEAssetsKeyPrefix=$(S3_ASSET_PREFIX) \
		--parameters FoundationStack:EETeamRoleArn=$(TEAM_ASSUMED_ROLE_ARN) \
		--parameters FoundationStack:SourceZipFile=$(ZIPFILE) \
		--parameters FoundationStack:SourceZipFileChecksum=$$(openssl sha256 -hex -r $(ZIPFILE) | cut -d' ' -f1) 

.PHONY: destroy
destroy:
	npx cdk destroy FoundationStack --force
