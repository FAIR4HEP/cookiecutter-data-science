.PHONY: clean data lint requirements download_data

#################################################################################
# GLOBALS                                                                       #
#################################################################################

PROJECT_DIR := $(shell dirname $(realpath $(lastword $(MAKEFILE_LIST))))
PROJECT_NAME = {{ cookiecutter.repo_name }}
DATA_DOI = {{ cookiecutter.data_doi }}
CODE_DOI = {{ cookiecutter.code_doi }}
PROVIDE_DOCKER = {{ cookiecutter.provide_dockerfile }}
{% if 'kaggle' in cookiecutter.data_doi.lower() %}
DATASET_NAME := $(shell curl -Ls -o /dev/null -w %{url_effective} https://doi.org/$(DATA_DOI) | awk -F / '{print $$5"/"$$6}')
{% endif %}

ifeq (,$(shell which conda))
HAS_CONDA=False
else
HAS_CONDA=True
endif

#################################################################################
# COMMANDS                                                                      #
#################################################################################

## Install Python Dependencies
requirements: test_environment
	python3 -m pip install -U pip setuptools wheel
	python3 -m pip install -r requirements.txt

## Make Dataset
data: requirements
	download_data
	python3 src/data/make_dataset.py data/raw data/processed

## Delete all compiled Python files
clean:
	find . -type f -name "*.py[co]" -delete
	find . -type d -name "__pycache__" -delete

## Lint using flake8
lint:
	flake8 src

{% if 'zenodo' in cookiecutter.data_doi.lower() %}
## Dowload Data from Zenodo with DOI
download_data:
	zenodo_get $(DATA_DOI) -o data/raw
{% elif 'opendata' in cookiecutter.data_doi.lower() %}
## Dowload Data from CERN Open Data Portal with DOI
download_data:
	cd data/raw && cernopendata-client download-files --doi $(DATA_DOI) && cd ../..
{% elif 'kaggle' in cookiecutter.data_doi.lower() %}
## Dowload Data from Kaggle with dataset name
download_data:
	cd data/raw && kaggle datasets download $(DATASET_NAME) && cd ../..
{% else %}
download_data:
	echo "Implement download of data"
{% endif %}

## Set up python interpreter environment
create_environment:
ifeq (True,$(HAS_CONDA))
	@echo ">>> Detected conda, creating conda environment."
	conda create --name $(PROJECT_NAME) python=3
	@echo ">>> New conda env created. Activate with:\nsource activate $(PROJECT_NAME)"
else
	python3 -m pip install -q virtualenv virtualenvwrapper
	@echo ">>> Installing virtualenvwrapper if not already installed.\nMake sure the following lines are in shell startup file\n\
	export WORKON_HOME=$$HOME/.virtualenvs\nexport PROJECT_HOME=$$HOME/Devel\nsource /usr/local/bin/virtualenvwrapper.sh\n"
	@bash -c "source `which virtualenvwrapper.sh`;mkvirtualenv $(PROJECT_NAME) --python=python3"
	@echo ">>> New virtualenv created. Activate with:\nworkon $(PROJECT_NAME)"
endif

## Test python environment is setup correctly
test_environment:
	python3 test_environment.py

#################################################################################
# PROJECT RULES                                                                 #
#################################################################################



#################################################################################
# Self Documenting Commands                                                     #
#################################################################################

.DEFAULT_GOAL := help

# Inspired by <http://marmelab.com/blog/2016/02/29/auto-documented-makefile.html>
# sed script explained:
# /^##/:
# 	* save line in hold space
# 	* purge line
# 	* Loop:
# 		* append newline + line to hold space
# 		* go to next line
# 		* if line starts with doc comment, strip comment character off and loop
# 	* remove target prerequisites
# 	* append hold space (+ newline) to line
# 	* replace newline plus comments by `---`
# 	* print line
# Separate expressions are necessary because labels cannot be delimited by
# semicolon; see <http://stackoverflow.com/a/11799865/1968>
.PHONY: help
help:
	@echo "$$(tput bold)Available rules:$$(tput sgr0)"
	@echo
	@sed -n -e "/^## / { \
		h; \
		s/.*//; \
		:doc" \
		-e "H; \
		n; \
		s/^## //; \
		t doc" \
		-e "s/:.*//; \
		G; \
		s/\\n## /---/; \
		s/\\n/ /g; \
		p; \
	}" ${MAKEFILE_LIST} \
	| LC_ALL='C' sort --ignore-case \
	| awk -F '---' \
		-v ncol=$$(tput cols) \
		-v indent=19 \
		-v col_on="$$(tput setaf 6)" \
		-v col_off="$$(tput sgr0)" \
	'{ \
		printf "%s%*s%s ", col_on, -indent, $$1, col_off; \
		n = split($$2, words, " "); \
		line_length = ncol - indent; \
		for (i = 1; i <= n; i++) { \
			line_length -= length(words[i]) + 1; \
			if (line_length <= 0) { \
				line_length = ncol - indent - length(words[i]) - 1; \
				printf "\n%*s ", -indent, " "; \
			} \
			printf "%s ", words[i]; \
		} \
		printf "\n"; \
	}' \
	| more $(shell test $(shell uname) = Darwin && echo '--no-init --raw-control-chars')
