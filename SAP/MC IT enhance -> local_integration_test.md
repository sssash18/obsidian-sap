#Issue 
### Acheivements

- The IT test `make test-integration` target is now capable of running both seed as control and a shoot as a control cluster.

### Changes

- Prompts the user to go on either way.
- MCM is now cloned also in this script at `dev/mcm`
- If user chooses shoot as control plane, the script does all the work of applying secrets, etc.
