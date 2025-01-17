# Options passed on the command line
ERROR_ON_FAILURE=1
V = 0
CAT_ALL_LOGS=0

####

DEFAULT_TESTS = 00 01 02 03 04 05 07 08 08-2 08-3 09 11 12 14 15 16 17 19 21 22 23 24 25 28 29 30 31 35 36 37 38 39 40 41 42 43 45 47 50
CONDITIONAL_TESTS = 06 10 13 18 20 26 27 32 33 34 44 46 48 49

ONLY_IF_COQTOP_COMPILE_TESTS =
ONLY_IF_Q_TESTS = 10 26
ONLY_IF_W_TESTS = 44 # 46 (even more special handling due to version constraints)
ONLY_IF_FROM_TESTS = 13 27 32 33 34
ONLY_IF_GT_84 = 20 49

MAIN_RUNS = $(patsubst %,run-example-%.sh,$(DEFAULT_TESTS) $(CONDITIONAL_TESTS) $(BROKEN_TESTS))
OTHER_RUNS = run-example-08.py
IGNORE = prefix-grep.py disable-parallel-make.sh init-settings.sh memoize.py

DEFAULT_LOGS = $(patsubst %,example_%_result.log,$(DEFAULT_TESTS))
CONDITIONAL_LOGS = $(patsubst %,example_%_result.log,$(CONDITIONAL_TESTS))
ONLY_IF_COQTOP_COMPILE_LOGS = $(patsubst %,example_%_result.log,$(ONLY_IF_COQTOP_COMPILE_TESTS))
ONLY_IF_Q_LOGS = $(patsubst %,example_%_result.log,$(ONLY_IF_Q_TESTS))
ONLY_IF_W_LOGS = $(patsubst %,example_%_result.log,$(ONLY_IF_W_TESTS))
ONLY_IF_FROM_LOGS = $(patsubst %,example_%_result.log,$(ONLY_IF_FROM_TESTS))
ONLY_IF_GT_84_LOGS = $(patsubst %,example_%_result.log,$(ONLY_IF_GT_84_TESTS))
ENABLED_LOGS = $(DEFAULT_LOGS)

.DEFAULT_GOAL := test-suite
.SECONDEXPANSION:

# Order the builds for 8 so they don't run in parallel
example_08-2_result.log: | example_08_result.log
example_08-3_result.log: | example_08-2_result.log

# some makefile magic to make things less noisy unless you do `make V=1`
Q_0 := @
Q_1 :=
Q = $(Q_$(V))

VECHO_0 := @echo
VECHO_1 := @true
VECHO = $(VECHO_$(V))

VECHON_0 := @echo -n
VECHON_1 := @true
VECHON = $(VECHON_$(V))

# inner vecho, without the @
VIECHO_0 := echo
VIECHO_1 := true
VIECHO = $(VIECHO_$(V))

RED=\033[0;31m
# No Color
NC=\033[0m
GREEN=\033[0;32m
YELLOW=\033[0;33m
BOLD=$(shell tput bold)
NORMAL=$(shell tput sgr0)

COQC_VERSION:=$(subst version ,,$(shell coqc --version 2>&1 | grep -o 'version [^ ]*'))
COQC_SUPPORTS_Q:=$(shell coqc --help 2>&1 | grep -c -- -Q)
COQTOP_SUPPORTS_COMPILE:=$(shell coqtop --help 2>&1 | tr '\n' '~' | sed s'/Deprecated options.*//g' | tr '~' '\n' | grep -c -- '\s-compile\s')
COQC_SUPPORTS_FROM:=$(shell echo "From Coq.Init Require Import Notations." | coqtop 2>&1 | grep -c 'Error')
COQC_SUPPORTS_W:=$(shell coqtop --help 2>&1 | grep -c -- '-w ')

.PHONY: check
check::
	@echo -n "Running with PYTHON=$(GREEN)"
	@bash -c '. init-settings.sh; echo -n "$${PYTHON}"'
	@echo -n "$(NC) ($(GREEN)"
	@bash -c '. init-settings.sh; echo -n "$$(which "$${PYTHON}")"'
	@echo "$(NC)) (based on $(GREEN)$${PYTHON}$(NC))"
	$(MAKE) -B test-suite

.PHONY: print-support
print-support::

ifeq (1,$(CAT_ALL_LOGS))
check::
	$(MAKE) test-suite-verbose-report
else
check::
	$(MAKE) test-suite-report
endif

.PHONY: test-suite
test-suite: $(DEFAULT_LOGS)

ifneq (0,$(strip $(COQTOP_SUPPORTS_COMPILE)))
test-suite: $(ONLY_IF_COQTOP_COMPILE_LOGS)
ENABLED_LOGS += $(ONLY_IF_COQTOP_COMPILE_LOGS)
print-support::
	@printf "coqtop supports -compile:\t\tYes\n"
else
print-support::
	@printf "coqtop supports -compile:\t\tNo\n"
endif

ifneq (0,$(strip $(COQC_SUPPORTS_Q)))
test-suite: $(ONLY_IF_Q_LOGS)
ENABLED_LOGS += $(ONLY_IF_Q_LOGS)
print-support::
	@printf "coqc supports -Q:\t\t\tYes\n"
else
print-support::
	@printf "coqc supports -Q:\t\t\tNo\n"
endif

ifneq (0,$(strip $(COQC_SUPPORTS_W)))
test-suite: $(ONLY_IF_W_LOGS)
ENABLED_LOGS += $(ONLY_IF_W_LOGS)
print-support::
	@printf "coqc supports -w:\t\t\tYes\n"
else
print-support::
	@printf "coqc supports -w:\t\t\tNo\n"
endif

ifeq (0,$(strip $(COQC_SUPPORTS_FROM)))
test-suite: $(ONLY_IF_FROM_LOGS)
ENABLED_LOGS += $(ONLY_IF_FROM_LOGS)
print-support::
	@printf "coqc supports From ... Require ...:\tYes\n"
else
print-support::
	@printf "coqc supports From ... Require ...:\tNo\n"
endif

ifeq (,$(strip $(filter 8.3% 8.4%,$(COQC_VERSION))))
test-suite: $(ONLY_IF_GT_84_LOGS)
ENABLED_LOGS += $(ONLY_IF_GT_84_LOGS)
print-support::
	@printf "coqc is 8.3 or 8.4:\t\t\tNo\n"
else
print-support::
	@printf "coqc is 8.3 or 8.4:\t\t\tYes\n"
endif

ifeq (,$(strip $(filter 8.3% 8.4% 8.5% 8.6% 8.7% 8.8% 8.9% 8.10% 8.11% 8.12% 8.13% 8.14%,$(COQC_VERSION))))
ifneq (0,$(strip $(COQC_SUPPORTS_W)))
test-suite: example_46_result.log
ENABLED_LOGS += example_46_result.log
endif
print-support::
	@printf "coqc is < 8.15:\t\t\tNo\n"
else
print-support::
	@printf "coqc is < 8.15:\t\t\tYes\n"
endif

ifeq (,$(strip $(filter 8.3% 8.4% 8.5% 8.6% 8.7% 8.8% 8.9% 8.10% 8.11%,$(COQC_VERSION))))
test-suite: example_48_result.log
ENABLED_LOGS += example_48_result.log
print-support::
	@printf "coqc is < 8.12:\t\t\tNo\n"
else
print-support::
	@printf "coqc is < 8.12:\t\t\tYes\n"
endif

$(DEFAULT_LOGS) $(CONDITIONAL_LOGS) : example_%_result.log : $$(wildcard example_%/example_%.v) run-example-%.sh
	$(VECHO) "TEST run-example-$*.sh..." >&2
	$(Q)yes "y" 2>/dev/null | ./run-example-$*.sh >"example_$*_make.log" 2>&1; RV=$$?; (echo "$$RV" > "$@")
	$(Q)RV="$$(cat "$@")"; if [ "$$RV" -eq 0 ]; then RESULT="$(GREEN)success!$(NC)"; else RESULT="$(RED)failure$(NC)"; fi; \
	echo "TEST run-example-$*.sh... $$RESULT" >&2

test-suite-verbose-report::
	$(Q)$(MAKE) test-suite-report -k
	$(Q)ALL="$(sort $(wildcard $(ENABLED_LOGS)))"; \
	for i in $$ALL; do                             \
	        if [ "$$(cat $$i)" -ne 0 ]; then       \
	                echo "$(RED)$$i$(NC):";        \
	        else                                   \
	                echo "$(GREEN)$$i$(NC):";      \
	        fi;                                    \
	        cat "$${i%%_result.log}_make.log";     \
	done

test-suite-verbose-report test-suite-report::
	$(Q)ALL="$(sort $(ENABLED_LOGS))";                               \
	SUCCESS=;                                                        \
	FAILURE=;                                                        \
	UNKNOWN=;                                                        \
	for i in $$ALL; do                                               \
	        if [ -f "$$i" ]; then                                    \
	                if [ "$$(cat $$i)" -ne 0 ]; then                 \
	                        FAILURE="$$FAILURE $$i";                 \
	                else                                             \
	                        SUCCESS="$$SUCCESS $$i";                 \
	                fi;                                              \
	        else                                                     \
	                UNKNOWN="$$UNKNOWN $$i";                         \
	        fi;                                                      \
	done;                                                            \
	if [ ! -z "$$FAILURE" ]; then                                    \
	        echo "$(BOLD)Failing test logs:$(NORMAL)";               \
	        for i in $$FAILURE; do                                   \
	                echo "$(RED)$$i$(NC):";                          \
	                cat "$${i%%_result.log}_make.log";               \
	        done;                                                    \
	fi;                                                              \
	if [ ! -z "$$SUCCESS" ]; then                                    \
	        echo "$(BOLD)Passing tests:$(NORMAL)";                   \
	        for i in $$SUCCESS; do                                   \
	                echo "$(GREEN)$$i$(NC)";                         \
	        done;                                                    \
	fi;                                                              \
	if [ ! -z "$$UNKNOWN" ]; then                                    \
	        echo "$(BOLD)Unknown tests:$(NORMAL)";                   \
	        for i in $$UNKNOWN; do                                   \
	                echo "$(YELLOW)$$i$(NC)";                        \
	        done;                                                    \
	fi;                                                              \
	if [ ! -z "$$FAILURE" ]; then                                    \
	        echo "$(RED)$(BOLD)Failing tests:$(NORMAL)$(NC)";        \
	        for i in $$FAILURE; do                                   \
	                echo "$(RED)$$i$(NC)";                           \
	        done;                                                    \
		exit $(ERROR_ON_FAILURE);                                \
	elif [ -z "$$UNKNOWN" ]; then                                    \
	        echo "$(GREEN)$(BOLD)All tests succeeded$(NORMAL)$(NC)"; \
	fi





MISSING_FILES = $(filter-out $(MAIN_RUNS) $(OTHER_RUNS) $(IGNORE), $(wildcard *.sh *.py))
.PHONY: has-all-tests
ifneq (,$(MISSING_FILES))
has-all-tests::
	@echo "$(RED)Missing files: $(MISSING_FILES)$(NC)"
	@exit 1
else
has-all-tests::

endif

clean::
	rm -f */bug*.v */*.vo */*.v.d */*.glob $(DEFAULT_LOGS) $(CONDITIONAL_LOGS)
