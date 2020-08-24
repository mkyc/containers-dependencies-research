VERSION:=1
USER:=mnemos
IMAGE:=epi-c-bmk

.PHONY: build

build: guard-VERSION guard-IMAGE guard-USER
	docker build -t $(USER)/$(IMAGE):$(VERSION) .

guard-%:
	@ if [ "${${*}}" = "" ]; then \
		echo "Environment variable $* not set"; \
		exit 1; \
	fi
