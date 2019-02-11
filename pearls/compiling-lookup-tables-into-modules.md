# Compiling lookup tables in Modules

The postgrex package has an interesting text file called `errcodes.txt`. Here is
a snippet from that file:

```
#
# errcodes.txt
#      PostgreSQL error codes
#
# Copyright (c) 2003-2015, PostgreSQL Global Development Group

# ...

Section: Class 00 - Successful Completion

00000    S    ERRCODE_SUCCESSFUL_COMPLETION                                  successful_completion

Section: Class 01 - Warning

# do not use this class for failure conditions
01000    W    ERRCODE_WARNING                                                warning
0100C    W    ERRCODE_WARNING_DYNAMIC_RESULT_SETS_RETURNED                   dynamic_result_sets_returned
01008    W    ERRCODE_WARNING_IMPLICIT_ZERO_BIT_PADDING                      implicit_zero_bit_padding
01003    W    ERRCODE_WARNING_NULL_VALUE_ELIMINATED_IN_SET_FUNCTION          null_value_eliminated_in_set_function
01007    W    ERRCODE_WARNING_PRIVILEGE_NOT_GRANTED                          privilege_not_granted
01006    W    ERRCODE_WARNING_PRIVILEGE_NOT_REVOKED                          privilege_not_revoked
01004    W    ERRCODE_WARNING_STRING_DATA_RIGHT_TRUNCATION                   string_data_right_truncation
01P01    W    ERRCODE_WARNING_DEPRECATED_FEATURE                             deprecated_feature

# ...

```

This file maps error codes to their symbols. The reason this was in the `lib`
folder was because it was supposed to be used as a source for error code
mapping. Upon further reading I found that this was being used in a module
called `Postgrex.ErrorCode`. Here are the interesting pieces of that module:

```elixir
defmodule Postgrex.ErrorCode do
  @external_resource errcodes_path = Path.join(__DIR__, "errcodes.txt")

  errcodes = for line <- File.stream!(errcodes_path),
             # ...

  # errcode duplication removal

  # defining a `code_to_name` function for every single error code which maps
  # the code to a name.
  for {code, errcodes} <- Enum.group_by(errcodes, &elem(&1, 0)) do
    [{^code, name}] = errcodes
    def code_to_name(unquote(code)), do: unquote(name)
  end
  def code_to_name(_), do: nil

end
```

This code uses our `errorcodes.txt` text file to define around 400 functions
which embed the actual *code to name mapping*. And whenever you wanted to do the
actual lookup you could just use `Postgrex.ErrorCode.code_to_name(error_code)`

[TODO other instances of this (elixir unicode)]

## Custom implementation

## Links

  - [Postgrex GitHub Repo](https://github.com/elixir-ecto/postgrex)
  - [TODO link errorcodes.txt](https://github.com/elixir-ecto/postgrex/blob/master/lib/postgrex/binary_utils.ex)
  - [TODO link errorcodes.ex](https://github.com/elixir-ecto/postgrex/blob/master/lib/postgrex/binary_utils.ex)


## Tags

  - #meta-programming
  - #performance
  - #lookup-tables
