#Makefile
ifdef OS
   BLAT = $(powershell  -noprofile rm .\.terraform\ -force -recurse)
   RM = $(powershell  -noprofile rm .\.terraform\ -force -recurse)
else
   ifeq ($(shell uname), Linux)
      RM  = rm .terraform/modules/ -fr
      BLAT= rm .terraform/ -fr
   endif
endif

.PHONY: all

all: init plan build

init:
	$(RM)
	terraform init -reconfigure -upgrade

plan:
	terraform plan --out tfplan.binary -refresh=true

planned: init
	terraform plan -out tf.plan
	terraform show -json tf.plan  > tf.json
	checkov -f tf.json

p:
	terraform plan -refresh=true

apply: build

build:
	terraform apply -auto-approve

convert:
	terraform show -json tfplan.binary > tfplan.json

check: init
	terraform plan -detailed-exitcode

destroy: init
	terraform destroy -force

docs:
	terraform-docs md . > README.md

valid:
	terraform fmt -recursive
	checkov -d . --external-checks-dir ../../checkov
	tfsec .
	terrascan scan

conftest: plan convert
	conftest test ./tfplan.json -p ../../policies

opa: plan convert
	opa eval --format pretty --data terraform.rego --input tfplan.json "data.terraform.analysis.authz"

target:
	@read -p "Enter Module to target:" MODULE;
	terraform apply -target $$MODULE

purge:
	$(BLAT)
