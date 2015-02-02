# fdb-conflicts

This is a small console app that reproduce the errors we get when commiting transaction in a concurrent fashion.

# the error

    FoundationDB error code 1025 (transaction_cancelled)

# when 

We get error code 1025 if we try to execute multiple concurrent non-conflicting transactions within a single go process at the moment we call `Commit`.

# scenario

This is how the code in this console app reproduces the errors:

* Open database
* Start a routine that writes 1000 static value batches to a channel called `values`
* Start 8 writers that read the values from that routines and write it to the database
* The writers write to the following key pattern: `global/{local_writer_id}/{local_writer_sequence}`

# run

Make sure you have the latest version of foundationdb client and server installed. Run the following command from the place where you cloned this repository:

    $ go get ./...
    $ GOMAXPROCS=6 go run main.go -v=1 -logtostderr
    started
    ...
    I0202 12:41:26.611009  125646 main.go:87] failed to commit transaction from 5: FoundationDB error code 1025 (transaction_cancelled)
    I0202 12:41:26.615375  125646 main.go:87] failed to commit transaction from 2: FoundationDB error code 1025 (transaction_cancelled)
    I0202 12:41:26.620152  125646 main.go:87] failed to commit transaction from 6: FoundationDB error code 1025 (transaction_cancelled)
    I0202 12:41:26.624472  125646 main.go:87] failed to commit transaction from 7: FoundationDB error code 1025 (transaction_cancelled)
    I0202 12:41:26.624561  125646 main.go:87] failed to commit transaction from 1: FoundationDB error code 1025 (transaction_cancelled)
    I0202 12:41:26.631456  125646 main.go:98] 6: finished, 91 success, 34 failed
    I0202 12:41:26.631477  125646 main.go:98] 4: finished, 95 success, 30 failed
    I0202 12:41:26.631477  125646 main.go:98] 5: finished, 91 success, 35 failed
    I0202 12:41:26.631486  125646 main.go:98] 2: finished, 93 success, 30 failed
    I0202 12:41:26.631491  125646 main.go:98] 7: finished, 91 success, 37 failed
    I0202 12:41:26.631491  125646 main.go:98] 0: finished, 94 success, 31 failed
    I0202 12:41:26.633323  125646 main.go:98] 3: finished, 94 success, 32 failed
    I0202 12:41:26.633361  125646 main.go:98] 1: finished, 94 success, 28 failed
