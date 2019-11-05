# Serverless Layers Bug

Layers can be utilised to deal with large dependencies. For example, this is utilised in the [`serverless-python-requirements` plugin.](https://www.npmjs.com/package/serverless-python-requirements#lambda-layer)

However, 
* *if* an already-large lambda is updated to use a layer for dependencies,
* *and* the dependencies are more than *half* the size limit for code and dependencies (250 MB),

the deployment may fail as both the *layer* and the *already-large* lambda are both applied simultaneously, exceeding the size limit.

This is demonstrated in this repo and `serverless` stack.

## Process to demonstrate issue

In each instance, the stack is being deployed via `sls deploy --stage <stage_name>`.

1. Update the `provider.cfnRole` option in the `serverless.yml` file.
2. Deploy tag `v0.1 - Initial, layer-less deployment`. This should deploy successfully, creating a single, large lambda with all dependencies bundled within it.
3. Deploy tag `v0.2 - Large lambda with layer`. This will not deploy due to size limitations, as the large layer is (presumably incorrectly) applied to the large lambda.
    * This is the bug being demonstrated.
4. Deploy tag `v0.3 - Small lambda without layer`. This will deploy successfully, creating a single, *small*, non-functional lambda *without* a layer attached, and a requirements layer.
    * This is a non-functional intermediate stage, required to deploy the layer without exceeding the file size limitation.
4. Deploy tag `v0.4 - Small lambda with layer`. This will deploy successfully, creating a single, *small*, functional lambda *with* a layer attached, and a requirements layer. 
    * This is the desired end-state.
