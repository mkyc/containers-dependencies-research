get-metadata: guard-C_TEMPLATES
	@echo \# AzI \| get-metadata-task \| should print component metadata
	@cat ${C_TEMPLATES}/mocked-metadata.yml

init: guard-C_TEMPLATES guard-C_SHARED guard-C_CONFIG
	@echo \# AzI \| init-task \| should create default config in shared directory
	@cp ${C_TEMPLATES}/mocked-config.yml ${C_SHARED}/${C_CONFIG}
	@if test -f ${C_SHARED}/state.yml ; \
	then \
		yq m -x -i ${C_SHARED}/state.yml ${C_TEMPLATES}/mocked-state-new.yml ; \
	else \
		yq m -x ${C_TEMPLATES}/mocked-state-new.yml > ${C_SHARED}/state.yml ; \
	fi

validate: guard-C_SHARED
	@echo \# AzI \| validate-task \| should validate if config file is correct
	@echo \# AzI \| validate-task \| should also be called automatically before apply
	@if test -f ${C_SHARED}/${C_CONFIG}; then yq v ${C_SHARED}/${C_CONFIG} ; else echo file not found && exit 1; fi

get-state: guard-C_SHARED
	@echo \# AzI \| get-state-task \| should output current state
	@if test -f ${C_SHARED}/state.yml; then cat ${C_SHARED}/state.yml ; else echo UNINITIALIZED && exit 1; fi

apply: guard-C_TEMPLATES guard-C_SHARED validate
	@echo \# AzI \| apply-task \| should apply module logic
	@yq m -x -i ${C_SHARED}/state.yml ${C_SHARED}/${C_CONFIG}
	@yq w -i ${C_SHARED}/state.yml azi.status applied
	$(eval last = $(shell yq r ${C_SHARED}/${C_CONFIG} azi.size))
	@number=0 ; while [[ $$number -lt $(last) ]] ; do \
		yq w -i ${C_SHARED}/state.yml azi.nodes[$$number].privateIP "10.0.0.$$number" ; \
		yq w -i ${C_SHARED}/state.yml azi.nodes[$$number].publicIP "213.1.1.$$number" ; \
		yq w -i ${C_SHARED}/state.yml azi.nodes[$$number].usedBy unused ; \
		number=`expr $$number + 1`; \
	done

guard-%:
	@ if [ "${${*}}" = "" ]; then \
		echo "Environment variable $* not set"; \
		exit 1; \
	fi
