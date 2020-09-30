
# ./jq
jq is a lightweight and flexible command-line JSON processor.

### Json to CSV
```bash
$ jq -r '.data[] | [.media.id, .media.name, .url] | @csv' ytblist.json > fileout2.csv
```
