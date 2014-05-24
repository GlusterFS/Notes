# Contribute to GlusterFS recipes

This guide details how to use issues and pull requests to improve GlusterFS recipes.

Please stick as close as possible to the guidelines. That way we ensure quality guides
and easy to merge requests.

Your Pull Request will be reviewed by one of our devs/volunteers and you will be
asked to reformat it if needed. We don't bite and we will try to be as flexible
as possible, so don't get intimidated by the extent of the guidelines :)

For better maintainance and clarity, some naming guidelines should be followed.
See details in each section below.

## Pull Request title

Try to be as more descriptive as you can in your Pull Request title.

## Guides

Each installation guide has its own namespace and it should be provided in a
`README` file so that it renders first when viewing the repository. Submit a new
one in `install/platform/README.md` (it doesn't have to be strictly in markdown though).

## Scripts

You are strongly encouraged to provide a `README` file that describes
how to use the script. You may have included all the needed info in the script
itself (recommended), so you could simply write something between the lines:

  > This script installs GlusterFS 3.5 on CentOS. Run it with `./centos.sh your_domain_name`
  >
  > For more info and variables you can change, read the comments in the script.


### Scripts doing similar things

There is a strong possibility that your script will do similar things to what a
script already in this repo do. In that case, please work on the existing script
and enhance it with your changes. No need to duplicate things.

## What information to put on your guide/script etc (mandatory)

If you have an installation guide to provide, fill in the template and place it on top
of it or include it in your installation script (commented), again on top. Try to
include as many items of this template as you can.

### Template

```
Distribution         : 
GlusterFS version    : 
Web Server           : 
Init system          : 
Database             : 
Contributors         : 
Additional Notes     : 
```
