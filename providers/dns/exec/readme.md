# Execute an external program

Solving the DNS-01 challenge using an external program.

## Description

The file name of the external program is specified in the environment variable `EXEC_PATH`.

When it is run by lego, three command-line parameters are passed to it:
The action ("present" or "cleanup"), the fully-qualified domain name and the value for the record.

For example, requesting a certificate for the domain 'foo.example.com' can be achieved by calling lego as follows:

```bash
EXEC_PATH=./update-dns.sh \
	lego --dns exec \
	--domains foo.example.com \
	--email invalid@example.com run
```

It will then call the program './update-dns.sh' with like this:

```bash
./update-dns.sh "present" "_acme-challenge.foo.example.com." "MsijOYZxqyjGnFGwhjrhfg-Xgbl5r68WPda0J9EgqqI"
```

The program then needs to make sure the record is inserted.
When it returns an error via a non-zero exit code, lego aborts.

When the record is to be removed again,
the program is called with the first command-line parameter set to `cleanup` instead of `present`.

If you want to use the raw domain, token, and keyAuth values with your program, you can set `EXEC_MODE=RAW`:

```bash
EXEC_MODE=RAW \
EXEC_PATH=./update-dns.sh \
	lego --dns exec \
	--domains foo.example.com \
	--email invalid@example.com run
```

It will then call the program `./update-dns.sh` like this:

```bash
./update-dns.sh "present" "foo.example.com." "--" "some-token" "KxAy-J3NwUmg9ZQuM-gP_Mq1nStaYSaP9tYQs5_-YsE.ksT-qywTd8058G-SHHWA3RAN72Pr0yWtPYmmY5UBpQ8"
```

## Commands

### Present

| Mode    | Command                                            |
|---------|----------------------------------------------------|
| default | `myprogram present -- <FQDN> <record>`             |
| `RAW`   | `myprogram present -- <domain> <token> <key_auth>` |

### Cleanup

| Mode    | Command                                            |
|---------|----------------------------------------------------|
| default | `myprogram cleanup -- <FQDN> <record>`             |
| `RAW`   | `myprogram cleanup -- <domain> <token> <key_auth>` |

### Timeout

The command have to display propagation timeout and polling interval into Stdout.

The values must be formatted as JSON, and times are in seconds.
Example: `{"timeout": 30, "interval": 5}`

If an error occurs or if the command is not provided:
the default display propagation timeout and polling interval are used.

| Mode    | Command                                            |
|---------|----------------------------------------------------|
| default | `myprogram timeout`                                |
| `RAW`   | `myprogram timeout`                                |


## NOTE

The `--` is because the token MAY start with a `-`, and the called program may try and interpret a - as indicating a flag.

In the case of urfave, which is commonly used,
you can use the `--` delimiter to specify the start of positional arguments, and handle such a string safely.
