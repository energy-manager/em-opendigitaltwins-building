# Changelog

The changelog contains all changes to the ontology. The purpose of the changelog is to have scrips for updating models in digital twins instances.

## Operations

### Adding

Use added when new models are added. The new models will not affect exiting models.

### Modifying

Use modified when exisiting models are modified. Digital twins models are immutable, such that a modification will either be to delete existing model and replace with new (keeping version/id), or to add a new version of the model. If adding a new version, then either depricate or delete old version.

If version/id is kept, then all twins using model will just refer to model. This approach should be used if the change is non-breaking, e.g. modifying a description or similar, such that the model semantics are kept.

If a change is breaking, then we should add a new model version. Twins using the old model should then be updated, and old model should be deleted. In some cases, the twins cannot be updated. Then the old model should be kept, but sat to be depricated. 

When creating a new model version, all references to old version in relationships must also be updated.

### Deleting

Deleting a model should only be done if no twins are using the model. Instead of deleting, use decommission to ensure that new twins will not use the model.

## Changelog syntax

The changelog should be formatted as:

    [
        {
            "comment": "Any comment",
            "ops": [
                <list of operations>
            ]
        }, ...
    ]

Operations will be formatted as:

Delete or decommission

    {
        "op": "delete" or "decommission",
        "model": <id of model, e.g. "dtmi:no:energymanager:agent:Agent;1">
    }

Add
                 
    {
        "op": "add",
        "file": <path to file, e.g. "Agent\\Agent.json">
    }

Patch

    {
        "op": "patch",
        "patch": <patch>
    }

Patch format follows [jsonPatch](https://jsonpatch.com/). Example with a new model which added a parameter can be:

    [
        {
            "op": "replace",
            "path": "/$metadata/$model",
            "value": "dtmi:example:foo_new;2"
        },
        {
            "op": "add",
            "path": "/temperature",
            "value": 60
        }
    ]

If relationships are changed by properties, then patch operation can be used to change properties of the relations.
If relationships are removed from a model, then the relationships should be deleted from the twins as well.
If relationships are changed by target, then relationships in twins should be deleted, but it will be impossible to know if new should be created, as they might represent something completely else.

# Final note

Updating models can break the twins, and queries related to the twins. Queries can be hardcoded to specific models and properties of relationships, making it hard to be replaced if there are changes that will affect this. E.g. if a model's relationship target is replaced, then it will be impossible to tell what the new target should point to of twins. Let's say you have two models, A and B, and a relationship "A points to B" in model A. If you now add a model C, and change model A's relationship to "A points to C" and in model C have "C points to B", then we cannot update the twins since there are no twins with model C.
