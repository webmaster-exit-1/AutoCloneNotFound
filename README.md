# AutoCloneNotFound
**ACNF** is a solution for packages not found in your GNU/Linux distro package/software repositories.

I had originally created this script while using Termux (terminal emulator) in Android.
I had cloned a project from Github then began to install the software dependencies,
which lead to many "error, package not found!" messages from Termux's repo.
So I went about creating this bash function and/or standalone method script
to aide with installing missing or "not found" software. 
9 times out of 10 if the software you need can't be found in your distro's repo(s)
It's probably going to be on Github. Note: This function/script
<u>does not</u> install software. It merely searches and clones the software.
Installing the software after the fact is still done by you, the end user.

You WILL need a Github personal api token to perform searches.
And you are subject to Github rate limiting you if your usage is deemed heavy on their servers.
That has to do with Github, not my code.
#
- > For Termux, add this to your bashrc which is located in: `$PREFIX/etc/bash.bashrc` <br>
   - > For PC GNU/Linux distro's your .bashrc can be found here: `$HOME/.bashrc`
#


- **ACNF** _.bashrc function_
  ```sh
  # Add this function to your .bashrc or a sourced script file

  install_or_clone() {
    local github_token="your_token_here" # Replace with your GitHub token

    # Function to install a package
    install_pkg() {
      local pkg_name=$1
      if pkg install -y "${pkg_name}"; then
        echo "${pkg_name} installed successfully."
      else
        echo "${pkg_name} not found in Termux repo. Searching on GitHub..."
        search_github_and_clone "${pkg_name}"
      fi
    }

    # Function to search GitHub and clone the repository
    search_github_and_clone() {
      local pkg_name=$1
      local search_result=$(curl -s -H "Authorization: token ${github_token}" "https://api.github.com/search/repositories?q=${pkg_name}+in:name&sort=stars&order=desc")
      local clone_url=$(echo "${search_result}" | grep -m 1 "clone_url" | cut -d '"' -f 4)

      if [ -n "${clone_url}" ]; then
        echo "Cloning ${clone_url}..."
        git clone "${clone_url}"
      else
        echo "No GitHub repository found for ${pkg_name}."
      fi
    }

    # Iterate over arguments passed to the function and attempt to install each one
    for pkg in "$@"; do
      install_pkg "${pkg}"
    done
  }
  ```

- **ACNF** _As a Standalone Script_
  ```bash
  #!/bin/bash

  ## These are example packages, replace with your own missing/"not found" packages ##
  # -------------------------------------------------------------------------------- #
  # List of packages you want to install
  packages=(
    "php7.4-curl" "hydra" "sqlmap" "nbtscan" "nikto" "whatweb" "sslscan" "jq" "golang" "adb" "xsltproc" "ldapscripts" "libssl-dev" "python-pip" "xvfb" "urlcrazy" "iputils-ping" "enum4linux" "dnsutils" # add the rest of your "package's not found" here
  )

  # GitHub personal access token for API authentication
  # Generate one at https://github.com/settings/tokens and replace "your_token_here" with it
  github_token="your_token_here"

  # Function to install a package
  install_pkg() {
    pkg_name=$1
    if pkg install -y "${pkg_name}"; then
      echo "${pkg_name} installed successfully."
    else
      echo "${pkg_name} not found in Termux repo. Searching on GitHub..."
      search_github_and_clone "${pkg_name}"
    fi
  }

  # Function to search GitHub and clone the repository
  search_github_and_clone() {
    pkg_name=$1
    # Using GitHub API to search repositories with the package name
    search_result=$(curl -s -H "Authorization: token ${github_token}" "https://api.github.com/search/repositories?q=${pkg_name}+in:name&sort=stars&order=desc")

    # Extracting the first repository's clone URL
    clone_url=$(echo "${search_result}" | grep -m 1 "clone_url" | cut -d '"' -f 4)

    if [ -n "${clone_url}" ]; then
      echo "Cloning ${clone_url}..."
      git clone "${clone_url}"
    else
      echo "No GitHub repository found for ${pkg_name}."
    fi
  }

  # Iterate over the list of packages and attempt to install each one
  for pkg in "${packages[@]}"; do
    install_pkg "${pkg}"
  done
  ```
  
