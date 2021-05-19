# Deploy Tools

This pipeline appears to be a mirror of GovWifi's primary deployment pipeline `deploy`, but with an unclear purpose.

We've lost the context for when and why it was created and by whom. 

The `deploy-tools` pipeline runs intermittently using the same inputs used in the `deploy` pipeline. It tries to pull secrets from Big Concourse which don't exist causing issues for the Automate team who maintain Big Concourse.

The GovWifi team agreed to delete the `deploy-tools` pipeline because it doesn't seem to serve a purpose. However, we're keeping a record of the configuration in case we need it in the future.
