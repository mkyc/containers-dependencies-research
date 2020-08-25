get-metadata: guard-C_TEMPLATES
	@echo \#get-metadata-task ---- should print component metadata
	@cat ${C_TEMPLATES}/mocked-metadata.yml

init: guard-C_TEMPLATES guard-C_SHARED guard-C_CONFIG
	@echo \#init-task ---- should create default config in shared directory
	@cp ${C_TEMPLATES}/mocked-config.yml ${C_SHARED}/${C_CONFIG}
	@if test -f ${C_SHARED}/state.yml ; \
	then \
		yq m -x -i ${C_SHARED}/state.yml ${C_TEMPLATES}/mocked-state-new.yml ; \
	else \
		yq m -x ${C_TEMPLATES}/mocked-state-new.yml > ${C_SHARED}/state.yml ; \
	fi
	$(eval unuseds = $(shell yq r ${C_SHARED}/state.yml --length --collect azi.nodes.usedBy==unused))
	$(eval required = $(shell yq r ${C_SHARED}/${C_CONFIG} bmk.size))
	$(eval unusedsPaths = $(shell yq r ${C_SHARED}/state.yml --printMode p azi.nodes.\(usedBy==unused\)))
	@export UP="$(unusedsPaths)" ; \
	if [ ${unuseds} -ge ${required} ] ; \
	then \
		echo OK ; \
		n0=0 ; n1=1; while [[ $$n0 -lt $(required) ]] ; do \
			c=`echo $$UP  | cut -d ' ' -f $$n1` ; \
			yq w -i ${C_SHARED}/${C_CONFIG} bmk.clusterNodes[$$n0].privateIP `yq r ${C_SHARED}/state.yml $$c.privateIP` ; \
			yq w -i ${C_SHARED}/${C_CONFIG} bmk.clusterNodes[$$n0].publicIP `yq r ${C_SHARED}/state.yml $$c.publicIP` ; \
			n0=`expr $$n0 + 1`; \
			n1=`expr $$n1 + 1`; \
		done ; \
	else \
		echo NOT ENOUGH UNUSED MACHINES ; \
	fi


validate: guard-C_SHARED
	@echo \#validate-task ---- should validate if config file is correct
	@echo \#validate-task ---- should also be called automatically before apply
	@if test -f ${C_SHARED}/${C_CONFIG}; then yq v ${C_SHARED}/${C_CONFIG} ; else echo file not found && exit 1; fi

get-state: guard-C_SHARED
	@echo \#get-state-task ---- should output current state
	@if test -f ${C_SHARED}/state.yml; then cat ${C_SHARED}/state.yml ; else echo UNINITIALIZED && exit 1; fi

apply: guard-C_TEMPLATES guard-C_SHARED validate
	@echo \#apply-task ---- should apply module logic
	@yq m -x -i ${C_SHARED}/state.yml ${C_SHARED}/${C_CONFIG}
	@yq w -i ${C_SHARED}/state.yml bmk.status applied
	@yq w -i ${C_SHARED}/state.yml bmk.clusterNodes.*.state created

guard-%:
	@ if [ "${${*}}" = "" ]; then \
		echo "Environment variable $* not set"; \
		exit 1; \
	fi
