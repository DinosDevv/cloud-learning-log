I learned about the difference between the windows powershell and the linux prompt. 

- I encountered failure in executing the SUDO command in the windows powershell, before switching over to the linux prompt by typing `wsl` and then `cd ~`

- I learned about what SSH keys are, create my own through the wsl prompt (in PowerShell) and added it to github as an authentication key

- I used the `ssh-keygen` command to generate my private access credential 

- I first failed creating the SSH key, with an error in the console that wrote `Saving key "cat ~/.ssh/id_ed25519.pub" failed: No such file or directory`, fixed it by entering the command `cd ~` to change the directory into the linux system (instead of the windows that I was previously in, despite running it through the linux prompt)

- I, then failed to test the key through github, with the message `The authenticity of host 'github.com (140.82.121.4)' can't be established.`, turned out to be a 2FA issue, that I resolved by simply passing the OTP into github.

