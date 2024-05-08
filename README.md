# AutoCloneNotFound
**ACNF** is a solution for packages not found in your GNU/Linux distro package/software repositories.

  I had originally created this script while using Termux (terminal emulator) in Android and I had cloned a project from Github then began to install the software dependencies,
which lead to many "error, package not found!" messages from Termux's repo.

  So I went about creating this bash function to aide with installing missing or "not found" software. <br>

9 times out of 10 times; If the software you need can't be found in your distro's repo(s).

It's probably going to be found on Github. <br>

Note: This function **DOES NOT** install software. It merely searches and clones the software. <br>

Installing the software after the fact is still done by you, the end user. <br>

You **WILL** need a Github personal api token to perform searches. <br>
And you are subject to Github rate limiting you if your usage is deemed heavy on their servers.
That has to do with Github, not my code.
#
- > For Termux, add this to your .bashrc which is located in: `$PREFIX/etc/bash.bashrc` <br>
   - > For PC GNU/Linux distro's your .bashrc can be found here: `$HOME/.bashrc`. And don't forget to edit `pkg` <-to-> `apt` or whichever package manager command your distro uses.
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

  
