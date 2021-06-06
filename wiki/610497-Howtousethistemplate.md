This template can be used in multiple ways.   Manual copying of this wiki template is strongly discouraged!  This sub wiki page should be deleted after it is copied.

- `challengeutils` - a Python/command line package that can be installed that has the ability to "create a challenge" - more documentation [here](https://sage-bionetworks.github.io/challengeutils/client/admin.html#create-a-challenge). 
- `synapseclient` - the [synapsePythonClient](https://python-docs.synapse.org/build/html/index.html) has a feature to copy a project to a new project.  Do not use the `synapseclient.copyWiki` command, make sure to use `synapseutils.copy`.  If you choose this option, there are multiple strings that you must edit in the template after you copy it to your new project.
    - home page
            - `teamId=0` with `teamId=[your team id}`
            - `#!Team:0` with `#!Team:{your team id}`
    - Participation Overview:
             - `teamId=0` with `teamId=[your team id}`
             - `#!Map:0` with `#!Map:{your team id}`
             - `challengeId=0` with `challengeId={your challenge id}`
    - Participants & Teams:
             - `teamId=0` with `teamId=[your team id}`
             - `challengeId=0` with `challengeId={your challenge id}`