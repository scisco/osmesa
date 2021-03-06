include config-aws.mk  # Variables for AWS options
include config-local.mk   # Variables related to running locally

# The osmesa container
LOCAL_IMG := osm_analytics:latest


#########
# LOCAL #
#########
docker-compose.local.yml:
	export LOCAL_IMG=${LOCAL_IMG}; \
	export AUGDIFF_SOURCE=${LOCAL_AUGDIFF_SOURCE}; \
	export CHANGESET_SOURCE=${LOCAL_CHANGESET_SOURCE}; \
	export CHANGE_SOURCE=${LOCAL_CHANGE_SOURCE}; \
	export AUGDIFF_START=${LOCAL_AUGDIFF_START}; \
	export CHANGESET_START=${LOCAL_CHANGESET_START}; \
	export CHANGE_START=${LOCAL_CHANGE_START}; \
		./expand.sh docker-compose.local.yml.tpl > docker-compose.local.yml

start-local: docker-compose.local.yml
	docker-compose -f docker-compose.local.yml up

stop-local:
	docker-compose -f docker-compose.local.yml down


#########
#  AWS  #
#########

login-aws-registry:
	eval `aws ecr get-login --no-include-email --region ${AWS_REGION}`

tag-image: login-aws-registry
	docker tag ${LOCAL_IMG} ${ECR_REPO}

push-image: login-aws-registry tag-image
	docker push ${ECR_REPO}

.PHONY: docker-compose.deploy.yml

docker-compose.deploy.yml: docker-compose.deploy.yml.tpl
	export ECR_REPO=${ECR_REPO}
	export AWS_LOG_GROUP=${AWS_LOG_GROUP}; \
	export AWS_REGION=${AWS_REGION}; \
	export AUGDIFF_SOURCE=${AUGDIFF_SOURCE}; \
	export AUGDIFF_START=${AUGDIFF_START}; \
	export CHANGESET_SOURCE=${CHANGESET_SOURCE}; \
	export CHANGESET_START=${CHANGESET_START}; \
	export DB_URI=${DB_URI}; \
		./expand.sh $< > $@

.PHONY: configure-cluster

configure-cluster:
	ecs-cli configure \
	  --cluster ${CLUSTER_NAME} \
	  --region ${AWS_REGION} \
	  --config-name ${CONFIG_NAME}

cluster-up:
	ecs-cli up \
	  --keypair ${KEYPAIR} \
	  --instance-role ${INSTANCE_ROLE} \
	  --security-group ${SECURITY_GROUP} \
	  --size 1 \
	  --instance-type ${INSTANCE_TYPE} \
	  --cluster-config ${CONFIG_NAME} \
	  --subnets ${SUBNETS} \
	  --vpc ${VPC} \
	  --force \
	  --verbose

cluster-down:
	ecs-cli down --cluster-config ${CONFIG_NAME}

.PHONY: create-service

create-service: docker-compose.deploy.yml configure-cluster
	ecs-cli compose \
	  --file $< create \
	  --cluster ${CLUSTER_NAME}

start-service: docker-compose.deploy.yml configure-cluster create-service
	ecs-cli compose --file $< service up \
	  --create-log-groups \
	  --cluster ${CLUSTER_NAME}

stop-service:
	ecs-cli compose down


#########
#  ALL  #
#########
build-container:
	cd .. && ./build-container.sh

clean:
	rm -f docker-compose.local.yml
	rm -f docker-compose.deploy.yml

