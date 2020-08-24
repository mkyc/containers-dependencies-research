ROOT_DIR:=$(shell dirname $(realpath $(firstword $(MAKEFILE_LIST))))

BMK_VERSION:=1

.PHONY: get-metadata init validate get-state apply clean

clean:
	rm $(ROOT_DIR)/shared/*


init-and-apply-kafka: guard-BMK_VERSION
	@docker run --rm \
		-v $(ROOT_DIR)/shared:/shared \
		-t mnemos/epi-c-bmk:$(BMK_VERSION) \
		init
	@docker run --rm \
		-v $(ROOT_DIR)/shared:/shared \
		-t mnemos/epi-c-bmk:$(BMK_VERSION) \
		apply

get-state: guard-VERSION guard-IMAGE guard-USER
	@docker run --rm \
		-v $(ROOT_DIR)/shared:/shared \
		-t mnemos/epi-c-bmk:$(BMK_VERSION) \
		get-state

guard-%:
	@ if [ "${${*}}" = "" ]; then \
		echo "Environment variable $* not set"; \
		exit 1; \
	fi
