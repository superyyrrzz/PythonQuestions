This is a doc on difference between Azure python SDK YAML format and DocFX's YAML format.

# Split YAML
## Now
namespace/class/method are in the same YAML’s `items`. This will generate a super long page like http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.operations.html
DocFX’s existing template generates a page for each class.
## Expected
Split to per class per YAML. Namespace YAML only has a item of type namespace, with classes in its `children`. Class YAML has items:
* First item: of type class
* Other items: of type method/field…

# Developer Comments
## Now
Summary contains a lot of structural information like param, type, overrides, rtype. See: https://github.com/bradygaster/python-sdk-dev/blob/eric-full-yaml-test/python-sdk-dev/docfx_yaml/azure.batch.operations.yml#L345
## Expected 
Map these info to YAML. Take this summary as an example, the ideal model is like:
```YAML
summary: Upgrades the operating system of the specified pool.
syntax:
  content: upgrade_os(pool_id, target_os_version, pool_upgrade_os_options=None, custom_headers=None, raw=False, **operation_config)
  Parameters:
-	id: pool_id
  type: str
  description: “The id of the pool to upgrade.”
  …
  return:
-	description: null
-	description: @"msrest.pipeline.ClientRawResponse?text=ClientRawResponse" if raw=true
```
Note that the `content` need to be added.
For return type, Multiple return type is allowed. Sphinx will create a link for return type if possible. It's like a uid with string as fallback in DocFX. The initial design in this snippet is to store it in a string `description`. This is different from .NET's YAML, which has both `type` and `description`. DocFX will try to resolve it as a uid first.
Note that part of the name can work in Sphinx, like `:vartype features: .operations.FeaturesOperations`. When filled into YAML's type, it should be a full uid like `type: azure.mgmt.resource.features.operations.FeaturesOperations`.

# Cross Reference
## Now
In rst, the cross reference is like `` :mytype:`myname<myuid>` ``.
## Expected
When converted to DFM, the syntax should be like `@"myuid?text=myname"`. As DocFX's cross reference is not aware of type, this can be omitted. See: http://www.sphinx-doc.org/en/stable/domains.html#python-roles
Question: seems Sphinx supports other tags like `:any:`, `:ref:`, and so on. Need they also be converted to markdown? see: http://www.sphinx-doc.org/en/stable/markup/inline.html#role-ref

# toc.yml
When generating metadata, DocFX will also generate a toc.yml reference to other YAMLs. This is also needed for sphinx-docfx-yaml.

# Other missing fields
## inheritance
In .NET, it's a list tracing to `System.Object` like:
``` yaml
inheritance:
- System.Object
- VBTestClass1.BaseClass1
```
In python, considering multiple Inheritance, it's also a lise of all it's base class:
``` yaml
inheritance:
- base1
- base2
...
```
