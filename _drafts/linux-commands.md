To recursively give directories read&execute privileges:

find /path/to/base/dir -type d -exec chmod 755 {} +
To recursively give files read privileges:

find /path/to/base/dir -type f -exec chmod 644 {} +
Or, if there are many objects to process:

chmod 755 $(find /path/to/base/dir -type d)
chmod 644 $(find /path/to/base/dir -type f)
Or, to reduce chmod spawning:

find /path/to/base/dir -type d -print0 | xargs -0 chmod 755 
find /path/to/base/dir -type f -print0 | xargs -0 chmod 644
