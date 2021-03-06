
HERE=${PWD}

PERLBREW_DIR=${HERE}/perlbrew
PERLBREW=export PERLBREW_ROOT=${PERLBREW_DIR} && PERLBREW_ROOT=${PERLBREW_DIR} ${PERLBREW_DIR}/bin/perlbrew
CPANM_DANCER1=${PERLBREW} exec --with dancer1_plugin_tests ${PERLBREW_DIR}/bin/cpanm
CPANM_DANCER2=${PERLBREW} exec --with dancer2_plugin_tests ${PERLBREW_DIR}/bin/cpanm
DZIL_DANCER2=${PERLBREW} exec --with dancer2_plugin_tests ${PERLBREW_DIR}/perls/dancer2_plugin_tests/bin/dzil

all: got_curl got_bash got_perlbrew_perl update_dancer2 plugins_list
	@echo "Plugin name,Pass on Dancer 1,Pass on Dancer 2" > result.csv
	@for i in `cat plugins_list`; do pass_d1=0; pass_d2=0; \
	  echo "* $$i"; \
          echo "  -> Dancer 1 deps"; ${CPANM_DANCER1} -n --installdeps $$i >/dev/null 2>&1 && echo "  -> Dancer 1 testing" && ${CPANM_DANCER1} --test-only $$i 2>&1 | grep 'Successfully tested' >/dev/null && pass_d1=1; \
          echo "  -> Dancer 2 deps"; ${CPANM_DANCER2} -n --installdeps $$i >/dev/null 2>&1 && echo "  -> Dancer 2 testing" && r=$$(DANCER_FORCE_PLUGIN_REGISTRATION=1 ${CPANM_DANCER2} --test-only -v $$i 2>&1); echo $$r | grep 'Successfully tested' >/dev/null && pass_d2=1; if (( pass_d2 )); then pass_d2='?'; echo $$r | grep DANCER2_PLUGIN_WERE_REGISTERED >/dev/null && pass_d2=1; fi;\
          echo "$$i,$$pass_d1,$$pass_d2" >> result.csv; \
        done;

clean:
	rm -rf plugins_list

fullclean:
	rm -rf ${PERLBREW_DIR}

${PERLBREW_DIR}/bin/perlbrew:
	@echo "* creating a local perlbrew"
	@export PERLBREW_ROOT=${PERLBREW_DIR} && curl -kL http://install.perlbrew.pl | bash
	@echo "* done"

${PERLBREW_DIR}/bin/cpanm:
	@echo "* installing cpanm"
	@${PERLBREW} install-cpanm
	@echo "* done"

got_curl:
	@which curl >/dev/null || ( echo "you don't have curl, please install it and retry" && false )

got_bash:
	@which bash >/dev/null || ( echo "you don't have bash, please install it and retry" && false )

got_perlbrew_perl: ${PERLBREW_DIR}/bin/perlbrew ${PERLBREW_DIR}/bin/cpanm perlbrew_dancer1_plugin_tests perlbrew_dancer2_plugin_tests

perlbrew_dancer1_plugin_tests:
	@${PERLBREW} list | grep dancer1_plugin_tests > /dev/null || ( ${PERLBREW} install -j 5 -n perl-5.16.1 --as dancer1_plugin_tests )

perlbrew_dancer2_plugin_tests:
	@${PERLBREW} list | grep dancer2_plugin_tests > /dev/null || ( ${PERLBREW} install -j 5 -n perl-5.16.1 --as dancer2_plugin_tests )

plugins_list:
	@echo "* fetching plugin lists"
	@cpanm -n Modern::Perl LWP::Simple ElasticSearch
	@${HERE}/get_modules_list.pl > plugins_list
	@echo "* done"

${PERLBREW_DIR}/perls/dancer2_plugin_tests/bin/dzil:
	@echo "* installing Dist::Zilla"
	@${CPANM_DANCER2} Dist::Zilla -n
	@echo "* done"

update_dancer2: ${PERLBREW_DIR}/perls/dancer2_plugin_tests/bin/dzil
	@echo "* updating Dancer2 from source"
	@cd ${HERE}/../.. && echo "  -> installing authordeps" && ( ${DZIL_DANCER2} authordeps --missing | grep -v dancer2_plugin_tests | grep -v '==========' | ( ${CPANM_DANCER2} -n ) ) >/dev/null && echo "  -> installing deps (this may take a while)" && ( ${DZIL_DANCER2} listdeps --missing | grep -v dancer2_plugin_tests | grep -v '==========' | ( ${CPANM_DANCER2} -n ) ) >/dev/null && echo "  -> installing Dancer 2" && ${DZIL_DANCER2} install --install-command='${CPANM_DANCER2} -v .' && cd ${HERE}
	@echo "* done"
