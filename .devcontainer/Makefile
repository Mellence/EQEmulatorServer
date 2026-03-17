# Build binaries: make cmake, make build
# One time initial setup (or to reset db): make prep, make inject-mariadb, make maps
# Update custom db edits: make inject-custom
# Start up server: make shared, make login, make world, make zone
# in game, stop combat spam #logs set gmsay 79 0
# in game, stop loot spam #logs set gmsay 69 0
NAME := eqemu-server

.PHONY: prep
prep: is-container
	@echo "Preparing build/bin for usage..."

	@echo "Prepping folders..."
	@mkdir -p .devcontainer/override
	@mkdir -p .devcontainer/repo
	@mkdir -p .devcontainer/cache
	@mkdir -p build/bin/logs
	@mkdir -p build/bin/shared
	@mkdir -p build/bin/assets
	@mkdir -p build/bin/logs/zone

	@echo "Applying overrides..."
	@if [ ! -f .devcontainer/override/eqemu_config.json ]; then cp .devcontainer/base/eqemu_config.json .devcontainer/override/eqemu_config.json; fi
	@sed -i -e "s/__DB_USER__/$(DB_USER)/g" -e "s/__DB_PASS__/$(DB_PASS)/g" -e "s/__DB_NAME__/$(DB_NAME)/g" .devcontainer/override/eqemu_config.json
	@if [ -f build/bin/eqemu_config.json ]; then unlink build/bin/eqemu_config.json; fi
	cd build/bin && ln -s ../../.devcontainer/override/eqemu_config.json eqemu_config.json
	@if [ ! -f .devcontainer/override/login.json ]; then cp .devcontainer/base/login.json .devcontainer/override/login.json; fi
	@if [ -f build/bin/login.json ]; then unlink build/bin/login.json; fi
	cd build/bin && ln -s ../../.devcontainer/override/login.json login.json

	@echo "Cloning repositories..."
	cd .devcontainer/repo && if [ ! -d "quests" ]; then cd ../../.devcontainer/repo/ && git clone https://github.com/ProjectEQ/projecteqquests.git quests; fi
	cd .devcontainer/repo && if [ ! -d "eqemu-definitions" ]; then cd ../../.devcontainer/repo/ && git clone https://github.com/xackery/eqemu-definitions.git eqemu-definitions; fi
	cd .devcontainer/repo && if [ ! -d "maps" ]; then cd ../../ && make maps; fi
	@if [ -d build/bin/quests ]; then unlink build/bin/quests; fi
	cd build/bin && ln -s ../../.devcontainer/repo/quests quests
	@if [ -d build/bin/maps ]; then unlink build/bin/maps; fi
	cd build/bin && ln -s ../../.devcontainer/repo/maps maps
	@if [ -d build/bin/eqemu-definitions ]; then unlink build/bin/eqemu-definitions; fi
	cd build/bin && ln -s ../../.devcontainer/repo/eqemu-definitions eqemu-definitions
	@mkdir -p build/bin/quests/mods

	@echo "Applying base links..."
	cp -R -u -p utils/patches .devcontainer/base/
	@if [ -d build/bin/assets/patches ]; then unlink build/bin/assets/patches; fi
	cd build/bin/assets && ln -s ../../../.devcontainer/base/patches patches
	@if [ -d build/bin/lua_modules ]; then unlink build/bin/lua_modules; fi
	cd build/bin && ln -s ../../.devcontainer/repo/quests/lua_modules lua_modules
	@if [ -d build/bin/mods ]; then unlink build/bin/mods; fi
	cd build/bin && ln -s ../../.devcontainer/repo/quests/mods mods
	@if [ -d build/bin/plugins ]; then unlink build/bin/plugins; fi
	cd build/bin && ln -s ../../.devcontainer/repo/quests/plugins plugins

	@echo "Eqemu is prepared. Edit build/bin/eqemu_config.json to configure."

is-container:
	@if [ ! -f /.dockerenv ]; then \
		echo "Not running in Docker container"; \
		exit 1; \
	fi

clean: is-container
	rm -rf build

.PHONY: maps
maps: is-container
	@echo "Downloading maps..."
	@mkdir -p .devcontainer/repo/maps
	@cd .devcontainer/repo/maps && wget -nc https://github.com/EQEmu/maps/archive/refs/heads/master.zip
	@cd .devcontainer/repo/maps && unzip -o master.zip
	@cd .devcontainer/repo/maps && mv maps-master/* .
	@cd .devcontainer/repo/maps && rm -rf maps-master
	@echo "Maps downloaded."

# Runs tests
.PHONY: test
test: is-container
	cd build/bin && ./tests

# Runs login binary
.PHONY: login
login: is-container check-mariadb
	cd build/bin && ./loginserver

.PHONY: hotfix
hotfix: shared

# Runs shared_memory binary
.PHONY: shared
shared: is-container check-mariadb
	cd build/bin && ./shared_memory

# Runs zone binary
.PHONY: zone
zone: is-container check-mariadb
	@find build/bin/logs/zone/ -type f -name 'zone*.log' -exec rm -f {} +
	cd build/bin && ./zone

check-mariadb: is-container
	@mariadb -h db -u $(DB_USER) -p$(DB_PASS) -e "SELECT 1" > /dev/null 2>&1 || (echo "Cannot reach MariaDB at db:3306"; exit 1)

.PHONY: set-expansion
set-expansion: is-container check-mariadb
	@mariadb -h db -u $(DB_USER) -p$(DB_PASS) --database $(DB_NAME) -e\
			"UPDATE rule_values SET rule_value='0' WHERE rule_name='Expansion:CurrentExpansion';"
	@echo "Expansion set to Classic (0)."

.PHONY: advance-expansion
advance-expansion: is-container check-mariadb
	@if [ -z "$(EXP)" ]; then \
			echo "Usage: make advance-expansion EXP=<number> (0=classic...8=OmensofWar)"; \
			exit 1; \
	fi
	@mariadb -h db -u $(DB_USER) -p$(DB_PASS) --database $(DB_NAME) -e \
			"UPDATE rule_values SET rule_value='$(EXP)' WHERE rule_name='Expansion:CurrentExpansion';"
	@echo "DB updated to expansion $(EXP). Log in as GM and run #reload rules global and #repopworld"

# Runs world binary
.PHONY: world
world: is-container check-mariadb
	@find build/bin/logs/ -type f -name 'world*.log' -exec rm -f {} +
	cd build/bin && ./world

# Runs ucs binary
.PHONY: ucs
ucs: is-container check-mariadb
	@find build/bin/logs/ -type f -name 'ucs*.log' -exec rm -f {} +
	cd build/bin && ./ucs

# Runs queryserv binary
.PHONY: queryserv
queryserv: is-container check-mariadb
	@find build/bin/logs/ -type f -name 'query_server*.log' -exec rm -f {} +
	cd build/bin && ./queryserv

reset-content:
	@echo "Resetting content tables in database peq..."
	cd .devcontainer/cache/db/peq-dump && sudo mariadb -h db -u $(DB_USER) -p$(DB_PASS) --database $(DB_NAME) -e "source create_tables_content.sql"

valgrind-%: is-container
	cd build/bin && valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes --verbose --log-file=logs/$*.valgrind.log ./$*

.PHONY: inject-mariadb
inject-mariadb:
	-mkdir -p .devcontainer/cache/db/
ifeq (,$(wildcard .devcontainer/cache/db/db.sql.zip))
	@echo ".devcontainer/cache/db.sql.zip not found. Downloading database from https://db.eqemu.dev/latest"
	wget -nc https://db.eqemu.dev/latest -O .devcontainer/cache/db/db.sql.zip
	-cd .devcontainer/cache/db && unzip db.sql.zip
endif
	@echo "Sourcing db may take a while, please wait..."

	@cd .devcontainer/cache/db/peq-dump && sudo mariadb -h db -u $(DB_USER) -p$(DB_PASS) --database $(DB_NAME) -e "source create_tables_content.sql"
	@cd .devcontainer/cache/db/peq-dump && sudo mariadb -h db -u $(DB_USER) -p$(DB_PASS) --database $(DB_NAME) -e "source create_tables_login.sql"
	@cd .devcontainer/cache/db/peq-dump && sudo mariadb -h db -u $(DB_USER) -p$(DB_PASS) --database $(DB_NAME) -e "source create_tables_player.sql"
	@# deprecated cd .devcontainer/cache/db/peq-dump && sudo mariadb -h db -u $(DB_USER) -p$(DB_PASS) --database $(DB_NAME) -e "source create_tables_queryserv.sql"
	@cd .devcontainer/cache/db/peq-dump && sudo mariadb -h db -u $(DB_USER) -p$(DB_PASS) --database $(DB_NAME) -e "source create_tables_state.sql"
	@cd .devcontainer/cache/db/peq-dump && sudo mariadb -h db -u $(DB_USER) -p$(DB_PASS) --database $(DB_NAME) -e "source create_tables_system.sql"
	@$(MAKE) set-expansion
	@echo "MariaDB is now injected."

.PHONY: gm-%
gm-%: is-container
	sudo mariadb -h db -u peq -p$(DB_PASS) --database peq -e "UPDATE account SET status=255 WHERE name = '$*';"
	@echo "Account $* is now a GM. /camp to have it go into effect."

depends: is-container
	sudo apt-get install graphviz pip time
	pip3 install graphviz
	mkdir -p build/depends
	@if [ ! -f "build/depends/dependency_graph.py" ]; then \
		wget https://raw.githubusercontent.com/pvigier/dependency-graph/master/dependency_graph.py -O build/depends/dependency_graph.py; \
	fi
	@echo "Generating dependency graphs (This may take a while)..."
	@echo "Login..."
	time python3 build/depends/dependency_graph.py -f png login build/depends/login.dot
	@echo "World..."
	time python3 build/depends/dependency_graph.py -f png world build/depends/world.dot
	@echo "Zone..."
	time python3 build/depends/dependency_graph.py -f png zone build/depends/zone.dot
	@echo "Common..."
	time python3 build/depends/dependency_graph.py -f png common build/depends/common.dot

backup: is-container
	@mkdir -p build/bin/backup
	cd build/bin && ./world database:dump --compress --player-tables --state-tables --system-tables --query-serv-tables

restore-%: is-container
	@if [ -z "$*" ]; then \
		echo "Please provide a backup file to restore from. Example: make restore-backup.sql"; \
		exit 1; \
	fi
	@echo "Restoring from backup $*"
	@sudo mariadb -h db -u peq -p$(DB_PASS) --database peq -e "$*"


cpu-zone: is-container
ifeq ($(findstring .devcontainer,$(CURDIR)),.devcontainer)
	@echo "This makefile is not intended to be run from the .devcontainer directory."
	exit
endif
	@cd build/bin && mkdir -p tmp
	cd build/bin && CPUPROFILE=prof.out ./zone

pprof-zone: is-container
ifeq ($(findstring .devcontainer,$(CURDIR)),.devcontainer)
	@echo "This makefile is not intended to be run from the .devcontainer directory."
	exit
endif
	cd build/bin && google-pprof --pdf zone prof.out > prof.pdf

pprof-gv-zone: is-container
ifeq ($(findstring .devcontainer,$(CURDIR)),.devcontainer)
	@echo "This makefile is not intended to be run from the .devcontainer directory."
	exit
endif
	cd build/bin && google-pprof --gv zone prof.out > prof.gv

heap-zone: is-container
ifeq ($(findstring .devcontainer,$(CURDIR)),.devcontainer)
	@echo "This makefile is not intended to be run from the .devcontainer directory."
	exit
endif
	@cd build/bin && mkdir -p tmp
	cd build/bin && HEAPPROFILE=prof.out ./zone


.PHONY: pull
pull:
	git pull
	@if [ ! -d "quests" ]; then git clone https://github.com/rebuildeq/quests.git quests; fi
	cd quests && git pull
	@if [ ! -d "eqemu-definitions" ]; then git clone https://github.com/xackery/eqemu-definitions.git eqemu-definitions; fi
	cd eqemu-definitions && git pull
