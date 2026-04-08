Thanks to https://stackoverflow.com/a/76122225/7915017
## Preparation
```
sudo apt install pip -y
python3 -m pip install --user rapidgzip --break-system-packages
```

Unfortunately pip install to some path which is not in PATH. So Ubuntu not find your application, let's add pip's folder to PATH
```
sudo bash -c 'cat <<EOF > "/etc/profile.d/local_bin_to_PATH.sh"
# Unfortunately pip install to some path which is not in PATH. So Ubuntu not find your application. 
# Note, \$ means that I want use string "$PATH", if I well be use just
#  export PATH="$PATH:/home/$SUDO_USER/.local/bin"
# then in file will be not good, like this (where q is example of corrent user name)
#  export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin:/home/q/.local/bin"
export PATH="\$PATH:/home/$SUDO_USER/.local/bin"
EOF'
```
To use new settings, log out, log in back or just make `source /etc/profile.d/local_bin_to_PATH.sh`

## Test
Make 1 MiB file with random info, put it to archive and remove original file. Ideally, you would have a file larger than 1 GB, or else, you will not see much benefit, depending on the number of cores.
```
head -c 1M /dev/urandom > sample.txt
tar -czvf "archive.tar.gz" sample.txt
rm sample.txt
```
Test rapidgzip
```
tar -x --use-compress-program=rapidgzip -f archive.tar.gz
```
Now you can find sample.txt file which was unarchived from the archive.

## Alias
Extract to the folder in the same path where archive is located.
```
cat <<'EOF' >> "$HOME/.bashrc"

untar() {
    local absolute_path="$1" # path to archive
    local archiveName=$(basename "$absolute_path")
    if [ -n "$2" ]; then # dir to unarchive is set by user
        local dir="$2"
    else # dir to unarchive will be set automatically based on archiveName
        local filename_no_extension="${archiveName%.*.*}"
        local dir="${2:-$filename_no_extension}"
    fi
    # printf "unarchive \"$archiveName\" to \"$dir\"" && [ -n "$3" ] && [ "$3" = "-sdel" ] && [ -f "archiveName" ] && printf " then trash \"$archiveName\"\n"
    case "$archiveName" in
        *.tar.gz) mkdir -p "$dir" && tar -x --use-compress-program=rapidgzip -f "$archiveName" --directory "$dir"
    esac
    [ -n "$3" ] && [ "$3" = "-sdel" ] && [ -f "archiveName" ] && gio trash "$archiveName" && echo "\"$archiveName\" trashed"
}
EOF
```
How to use? Example
```
untar archive.tar.gz myfolder222 -sdel
```
ToDo:
2) Switch logic based on tar format (bz2, gz, ...) like in https://gist.github.com/y0z/8b0337a0d8ba595d2b51bf883562f88b
