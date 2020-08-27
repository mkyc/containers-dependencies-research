ROOT_DIR:=$(shell dirname $(realpath $(firstword $(MAKEFILE_LIST))))

BMK_VERSION := 1
AZI_VERSION := 1

.PHONY: get-metadata init validate get-state apply clean

clean:
	rm $(ROOT_DIR)/shared/*

init-and-apply-azure-infrastructure: guard-AZI_VERSION
	@docker run --rm \
		-v $(ROOT_DIR)/shared:/shared \
		-t mnemos/epi-c-azi:$(AZI_VERSION) \
		init
	@docker run --rm \
		-v $(ROOT_DIR)/shared:/shared \
		-t mnemos/epi-c-azi:$(AZI_VERSION) \
		apply

init-and-apply-kafka: guard-BMK_VERSION
	@docker run --rm \
		-v $(ROOT_DIR)/shared:/shared \
		-t mnemos/epi-c-bmk:$(BMK_VERSION) \
		init
	@docker run --rm \
		-v $(ROOT_DIR)/shared:/shared \
		-t mnemos/epi-c-bmk:$(BMK_VERSION) \
		apply

state: guard-BMK_VERSION
	@if test -f $(ROOT_DIR)/shared/state.yml; then cat $(ROOT_DIR)/shared/state.yml ; else echo UNINITIALIZED && exit 1; fi

guard-%:
	@ if [ "${${*}}" = "" ]; then \
		echo "Environment variable $* not set"; \
		exit 1; \
	fi
