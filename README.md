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
tar -c -I 'zstd -22 --ultra --long -T0' -f "archive.zst" sample.txt
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
        *.tar.gz) local prog="--use-compress-program=rapidgzip";;
        *.tar.bz2) local prog="--use-compress-program=rapidgzip";;
        *.tar.lz) local prog="--use-compress-program=tarlz";;
        *.zst) local prog="--use-compress-program='unzstd -T0'";;
        *.tar.xz) local prog="--use-compress-program='xz -d -T0'";;
        *) echo "no parallel use-compress-program found, simple code will be used mkdir -p \""$dir"\" && tar xf \""$archiveName"\" --directory \""$dir"\"";;
    esac
    if [ -s "$archiveName" ]; then # check that file exists with size>0
        mkdir -p "$dir" && tar x $prog -f "$archiveName" --directory "$dir" || echo "err, mkdir -p \""$dir"\" && tar x $prog -f \""$archiveName"\" --directory \""$dir"\""
    else
        echo "err, archive !exists||empty, mkdir -p \""$dir"\" && tar x $prog -f \""$archiveName"\" --directory \""$dir"\""
    fi
    [ -n "$3" ] && [ "$3" = "-sdel" ] && [ -f "archiveName" ] && gio trash "$archiveName" && echo "\"$archiveName\" trashed"
}
EOF
```
How to use? Example
```
untar archive.tar.gz myfolder222 -sdel
untar archive_name.tar.xz
untar archive_name.zst
```
Known bugs:
1) untar archive_name.tar.xz should be fixed
1) untar archive_name.zst should be fixed
