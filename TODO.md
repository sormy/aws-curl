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
