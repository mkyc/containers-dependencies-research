get-metadata: guard-C_TEMPLATES
	@echo \#get-metadata-task ---- should print component metadata
	@cat ${C_TEMPLATES}/mocked-metadata.yml

init: guard-C_TEMPLATES guard-C_SHARED guard-C_CONFIG
	@echo \#init-task ---- should create default config in shared directory
	@cp ${C_TEMPLATES}/mocked-config.yml ${C_SHARED}/${C_CONFIG}
	@yq m -x ${C_TEMPLATES}/mocked-state-new.yml > ${C_SHARED}/state.yml

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
	@yq w -i ${C_SHARED}/state.yml bmk.clusterNodes.*.state new


guard-%:
	@ if [ "${${*}}" = "" ]; then \
		echo "Environment variable $* not set"; \
		exit 1; \
	fi
