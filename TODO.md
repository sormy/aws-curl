Here is a list of things that can be improved:

- use ranged uploads for uploading big files to s3
- support sending multiple requests using --next, -:
- recognize URL not only when passed as last (also as first or as --url)
- test all curl arguments and ban that are breaking the tool
- unit tests
- integ tests

url encode:

```sh
##
# Encodes string with url encoding.
# @see https://gist.github.com/cdown/1163649
# Arguments:
#   $1 string to encode
#   $2 exclude glob
# Returns:
#   url encoded string
##
urlencode() {
    str="$1"
    excludes="$2"

    old_lc_collate=$LC_COLLATE
    LC_COLLATE=C

    length="${#str}"
    for index in $(seq 1 "$length"); do
        char="$(expr substr "$str" "$index" 1)"
        case $char in
            $excludes) printf '%s' "$char" ;;
            [a-zA-Z0-9.~_-]) printf '%s' "$char" ;;
            *) printf '%%%02X' "'$char" ;;
        esac
    done

    LC_COLLATE=$old_lc_collate
}
```

Test encoding:

```
printf "\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0A\x0B\x0C\x0D\x0E\x0F\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1A\x1B\x1C\x1D\x1E\x1F\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2A\x2B\x2C\x2D\x2E\x2F\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3A\x3B\x3C\x3D\x3E\x3F\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4A\x4B\x4C\x4D\x4E\x4F\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5A\x5B\x5C\x5D\x5E\x5F\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6A\x6B\x6C\x6D\x6E\x6F\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7A\x7B\x7C\x7D\x7E\x7F"
```
