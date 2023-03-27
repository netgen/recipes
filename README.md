# Netgen Recipes

This repo holds all Symfony Flex recipes which can't be put into the official `symfony/recipes-contrib` repo
because of incompatible license like GPL.

## Configure access to recipes

Update your `composer.json` file with Flex endpoint:

```json
{
    "extra": {
        "symfony": {
            "endpoint": [
                "https://api.github.com/repos/netgen/recipes/contents/index.json?ref=flex",
                "flex://defaults"
            ]
        }
    }
}
```
