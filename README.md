fh ("For(each)Host")
====================

Run a command on multiple hosts (using ssh), in
parallel, collect the results, and print them in an easily parseable form.

## Installation

Simply download the `fh` script, put it somewhere on your path, and make it executable.

`fh` requires Python 2.x and OpenSSH, both of which should be already present on most posix-like systems.

It has not been tested on Windows, but may work provided you have appropriate command-line-accessible Python and ssh installed.  Some features (such as the interactive console) probably will not work without a full unix-like terminal environment. (If anyone wants to try it out, feedback/patches are welcome)

### Note:
This script can be insalled (i.e. symlinked) with the following alternate names, as shortcuts for specific behaviors:
  * fhcmd - Equivalent to `fh -c`
  * fhsh  - Equivalent to `fh --shell`
  * fhscp - Equivalent to `fh -pc scp`

## Syntax Summary
```
Usage: fh [options] command
  (Note: list of hosts is read from stdin by default)

Options:
  --version             show program's version number and exit
  --help                show this help message and exit
  --noconsole           Disable interactive console capability

  Job Options:
    -h HOSTLIST, --hosts=HOSTLIST
                        Use the specified host list, instead of reading from
                        stdin
    -f HOSTFILE, --file=HOSTFILE
                        Read host list from a file, instead of stdin
    -u USERNAME, --user=USERNAME
                        User to login to remote machines as
    -n NUMBER, --connections=NUMBER
                        Number of processes/connections to run in parallel
                        (default 20)
    -o SSHOPT, --ssh-option=SSHOPT
                        Pass an option setting to the ssh command
    -c, --command       Run command locally instead of using ssh
    --shell             Run command locally in a subshell (Note: May require
                        extra quoting)
    --nohostcheck       Don't check hostnames before starting
    -s FILENAME, --savefile=FILENAME
                        File to save results in (for use with --last)
    -l, --last          Print results from last run again (do not run a new
                        job)

  Output Options:
    -I, --immediate     Print output as soon as it's available (output from
                        different hosts may be intermingled)
    -W, --collect-output
                        Wait until all hosts have completed before printing
                        any results
    -O, --print-stdout  Print stdout from each host
    -E, --print-stderr  Print stderr from each host
    -S, --print-start   Print start time for each host
    -C, --print-completion
                        Print completion time and result code for each host
    -K, --print-killed  Print killed hosts (implied by -C)
    -A, --print-all     Print all types of output (equivalent to -OESCK)
    -T, --timestamp     Print a timestamp for each line
    -H HOSTLIST, --print-hosts=HOSTLIST
                        Only print the results from the specified host(s)
    -P, --parse-friendly
                        Produce output in a format easier to parse by shell
                        utilities
    -N, --noprefix      Do not prefix output lines with hostname/etc
    -L, --list          Only print hostnames, not other info
    --print-empty       Print a blank line for hosts which had no output
    --failed            Only print output from hosts with a nonzero return
                        status
    --succeeded         Only print output from hosts with a zero return status
    -p, --progress      Show progress information while running
    --noprogress        Do not show progress information
    --progress-stderr   Show progress information on stderr instead of tty
```

Note: When using --command or --shell, any occurrence of "%" in the command
will be replaced with the hostname (use "%%" for a literal percent character)

## Input

By default, `fh` will take a list of hostnames (or IP addressses) via stdin.  This makes it very convenient to, for example, run a command which produces a list of hosts, and then feed that list into `fh` to perform some function on them.

Alternately, hosts can be specified with the `--hosts` (`-h`) option instead.

Note that when reading from stdin, anything following a colon (`:`) on a line will be ignored.  This allows taking the output of one `fh` run, performing some processing on it (for example, grepping for particular text in the output) and then feeding the results directly back into `fh` to perform some command on the matching hosts (this can be particularly powerful in combination with `fh -l` (see below)).

## Re-viewing a previous run

Common scenario:  You run an `fh` command, get some output as a result, and then say "darn!  I wish I'd saved that output so I could do stuff with it without having to run that command on all those hosts again!"

Well, you're in luck!  Just run `fh -l` (aka `fh --last`)!  This will spit out the output of the last `fh` job you ran, without actually re-running anything on any hosts.  You can even give different output options if you want to get the results in a different format (for example, `fh -lL --failed` will print a list of the hosts which failed from the last run).

If you want to get really fancy, you can even use the `-s` option to specify a file to save the results in when you do an `fh` run, and then view it later with `fh -l -s <savefile>`

By default, `fh` uses a savefile called `.fh.last` in your home directory.  You can override this default by setting the `FH_SAVEFILE` environment variable to point to another location.

### PROTIP:  Per-terminal savefile

By default, `fh` uses the same savefile (`.fh.last`) for all runs.  This can get confusing if you have a bunch of terminal windows open and are doing different things in different ones, as running `fh` in one window can clobber the saved previous results of an `fh` run in another window.

You can fix this by setting `FH_SAVEFILE` to refer to a different file for each window.  For example, if you're using bash, you can put the following in your `.bashrc`:

    export FH_SAVEFILE="$HOME/.fh.last.$(tty | sed -e 's|^/dev/||' -e 's|/|_|g')"

This will cause `fh` to use savefiles like `.fh.last.pts_35` (where `pts_35` would be different for each window you open)

## Return Codes:
  0 = All hosts completed successfully

  1 = Some hosts completed with errors

  2 = There were problems with the input host list (or the process was
      interrupted) which may have prevented some intended hosts from being run.

  3 = There were fatal errors with the command parameters.  No hosts were run.

## Interactive Console

*Note: This is currently an experimental feature, and more documentation is on the way.*

When `fh` is running, if running on a terminal, you can press Ctrl-F to drop into a "console shell".  This does not interrupt `fh` (it will keep running) but allows you to issue commands to see the status of the run on various hosts, kill off particular hosts (if the command has stalled on them for some reason, for example), abort the whole run, etc.

For more information on the available commands, type `help` when at the `FH>` console prompt.

