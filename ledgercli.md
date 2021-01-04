# ledger-cli

`ledger-cli` is a plaintext accounting utility. The CLI processes Ledger
formatted plaintext files to perform accounting functions.

*    [ledger-cli.org](https://www.ledger-cli.org)
*    [ledger-cli.org - HTML Manual](https://www.ledger-cli.org/3.0/doc/ledger3.html)

## Basics

All commands require a ledger file provided using either the `-f` argument or the
`LEDGER_FILE` environment variable. I personally choose to use the env variable
as configured in my `.bashrc`:

    # .bashrc file
    LEDGER_FILE=/home/oliver/ledger.txt

Report account balances

    ledger bal Assets:Chequing
      CAD 100.00 Assets:Chequing

Report account balance history 

    ledger reg Assets:Chequing

Print transactions

    ledger print Assets:Chequing

List all accounts

    ledger accounts

List all payees

    ledger payees

List all commodities

    ledger commodities

## Data Mutator Flags

The following flags are used to mutate data values returned in variuos reports.

Convert commodities values to a common commodity:

    -X $COMMODITY

For example, convert account balances to CAD:

   ledger bal Assets -X CAD

It is best to be explicit with whether the conversion should be performed
with today's market price or historical cost values:

    # Historical expense costs
    ledger bal Expenses -H -X CAD

    # Current market values of assets
    ledger bal Assets -V -X CAD

Another complication is if you would like to know the market price on a
specific date, for example, end-of-year:

    # Historical market value of assets
    ledger bal Assets -V -X CAD --now 2019-12-31 --end 2019-12-31

## Sort Flags

Sort balances in decreasing order of total amount:

    ledger bal Expenses -S -T

## Output Flags

The following flags manipulate the output format of reports.

Report the full name of accounts rather than indicate nesting with indentation:

    ledger bal Assets --flat

Report only the date and output value (works for single account and commodity):

    ledger reg --amount-data -X CAD -H Assets:Chequing

## Filters

Limit to postings of a given commodity:

    ledger reg Assets -l 'commodity =~ /XBT/'

Print transactions for a given payee:

    ledger print Expenses -l 'payee =~ /KFC/'

Limit to certain date period:

    ledger print Assets:Chequing --begin 2020-12-01 --end 2020-12-31

Limit to specific tag:

    ledger reg %TFSA

Limit to negative postings (for example to report credit card spending):

    ledger bal "Liabilities:Credit Cards:TD" -l "amount < 0"

Limit to expenses including a specific account (to identify expenses charged
to credit card):

    ledger bal Expenses and expr 'any(account =~ /Liabilities:Credit Cards:PC/)'

## Custom Reports

View top expense accounts in decreasing amounts:

    ledger bal Expenses -H -X CAD --flat -S -T

Calculate total value of all assets in CAD:

    ledger bal Assets -X CAD -n

Show monthly asset value diff

    ledger -M register Assets -X CAD -n --no-rounding

Show monthly expense value diff

    ledger -M register Expenses -X CAD -n --no-rounding

Average increase in assets per month

    ledger reg Assets -X CAD -M -n --no-rounding --format "%t\n" | awk '{ sum += $2; n++ } END { if (n > 0) print sum / n; }'

View value held in each commodity reported in CAD:

    ledger bal Assets --group-by commodity -X CAD -V -n

