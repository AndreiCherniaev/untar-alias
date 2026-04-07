Thanks to https://stackoverflow.com/a/76122225/7915017
## Preparation
```
sudo apt install pip -y
python3 -m pip install --user rapidgzip --break-system-packages
```

Unfortunately pip install to some path which is not in PATH. So Ubuntu not find your application, let's add pip's folder to PATH
Ubuntu 24 and older
```
export myName=$USER && sudo -E bash -c 'cat <<EOF > /etc/profile.d/local_bin_to_PATH.sh
# Unfochently pip install to some path which is not in PATH. So Ubuntu not find your application. 
# Note, \$ means that I want use string "$PATH", if I well be use just
#  export PATH="$PATH:/home/$myName/.local/bin"
# then in file will be not good, like this (where q is example of corrent user name)
#  export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin:/home/q/.local/bin"
export PATH="\$PATH:/home/$myName/.local/bin"
EOF'
```
Ubuntu 25.10 and upper
```
sudo bash -c 'cat <<EOF > /etc/profile.d/local_bin_to_PATH.sh
# Unfochently pip install to some path which is not in PATH. So Ubuntu not find your application. 
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
    local absolute_path="$1" #path to archive
    local filename=$(basename "$absolute_path")
    local filename_no_extension="${filename%.*.*}"
    if [ -z "$2" ]; then
    echo "The second argument is empty or missing."
    fi
    local dir="${2:-$filename_no_extension}"
    # echo "unarchive $filename to $dir"
    case "$filename" in
        *.tar.gz) mkdir -p "$dir" && tar -x --use-compress-program=rapidgzip -f "$filename" --directory "$dir"
    esac
}
EOF
```
ToDo: 1) add ability to change base directory. For example I want untar from remote storage to local folder...
2) Switch logic based on tar format (bz2, gz, ...) like in https://gist.github.com/y0z/8b0337a0d8ba595d2b51bf883562f88b
