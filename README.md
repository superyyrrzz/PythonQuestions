This is a doc on difference between Azure python SDK YAML format and DocFX's YAML format.

Here's a YAML file generated by DocFX for reference: https://github.com/qinezh/op-docfx-seed/blob/master/api_ref/CatLibrary.Cat-2.yml. Generally we want Python's YAML to be as close as .NET's YAML.

# RST
We've investigated the rst-to-md tool that ASP.NET team uses. It still has some info loss. e.g.
* Table is dropped after converting to dfm.
* `.. warning::` is converted to plain text `warning:`, which is expected to converted to DFM's `> [!Warning]`

Another solution is to convert RST in comment directly to HTML. Some syntax that can't be converted directly can use DFM, like [cross reference](user-content-cross-reference), which will be discussed below.

DFM(DocFX Flavored Markdown) Specification: http://dotnet.github.io/docfx/spec/docfx_flavored_markdown.html

# Split YAML
## Now
Namespace/class/method are in the same YAML’s `items`. This will generate a super long page like http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.operations.html.
DocFX's existing template generates a page for each class.
## Expected
Split to per class per YAML. Namespace YAML only has a item of type namespace, with classes in its `children`. Class YAML has items:
* First item: of type class
* Other items: of type method/field…

# Developer Comments
## Now
Summary contains a lot of structural information like param, type, overrides, rtype. See: https://github.com/bradygaster/python-sdk-dev/blob/eric-full-yaml-test/python-sdk-dev/docfx_yaml/azure.batch.operations.yml#L345
## Expected 
Map these info to YAML. Take this summary as an example, the ideal model is like:
``` YAML
summary: Upgrades the operating system of the specified pool.
syntax:
  content: upgrade_os(pool_id, target_os_version, pool_upgrade_os_options=None, custom_headers=None, raw=False, **operation_config)
  parameters:
  - id: pool_id
    type: str
    description: "The id of the pool to upgrade."
  return:
  - description: null
  - description: @"msrest.pipeline.ClientRawResponse?text=ClientRawResponse" if raw=true
```
Note that the `content` need to be added.

For return type, Multiple return type is allowed. Sphinx will create a link for return type if possible. It's like a uid with string as fallback in DocFX. The initial design in this snippet is to store it in a string `description`. This is different from .NET's YAML, which has both `type` and `description`. DocFX will try to resolve it as a uid first.

Note that part of the name can work in Sphinx, like `:vartype features: .operations.FeaturesOperations`. When filled into YAML's type, it should be a full uid like `type: azure.mgmt.resource.features.operations.FeaturesOperations`.

# Cross Reference
## Now
In rst, the cross reference is like `` :mytype:`myname<myuid>` ``.
## Expected
- For internal reference, for example referencing `azure.graphrbac.operations.ObjectsOperations`, DocFX syntax is `@azure.graphrbac.operations.ObjectsOperations`. Another syntax is `@"azure.graphrbac.operations.ObjectsOperations?text=myname"` if display name is to be specified. DocFX's cross referencing is not dependent on type, but unique id, the following roles defined in http://www.sphinx-doc.org/en/stable/domains.html#python-roles could be omitted.
- For external reference, besides updating syntax with `@azure.graphrbac.operations.ObjectsOperations`, should also provide the `xrefmap` as follows, for the purpose of redirection. Refer to [Sample docfx.json](https://github.com/dotnet/docfx/blob/master/Documentation/docfx.json#L36) to download the xref map zip sample and how to specify the xref location in docfx.json. 
``` yaml
### YamlMime:XRefMap
sorted: true
hrefUpdated: true
references:
- uid: external.ref.str
  href: https://docs.python.org/3.5/library/stdtypes.html#str
  name: str
  fullName: str
```
Here's more info about DocFX's cross reference：http://dotnet.github.io/docfx/tutorial/links_and_cross_references.html

Question: seems Sphinx supports other tags like `:any:`, `:ref:`, and so on. Need they also be converted to markdown? see: http://www.sphinx-doc.org/en/stable/markup/inline.html#role-ref

# Toc.yml
When generating metadata, DocFX will also generate a toc.yml reference to other YAMLs. This is also needed for sphinx-docfx-yaml. Take following syntax as an example, more info please refer to [DocFx Table-Of-Content Tutorial](http://dotnet.github.io/docfx/tutorial/intro_toc.html)
``` yaml
### YamlMime:TableOfContent
- uid: azure.graphrbac.operations
  name: azure.graphrbac.operations
  items:
  - uid: azure.graphrbac.operations.ObjectsOperations
    name: ObjectsOperations
  - uid: azure.graphrbac.operations.ApplicationsOperations
    name: ApplicationsOperations
  - uid: azure.graphrbac.operations.GroupsOperations
    name: GroupsOperations
```

# Matching between documentation and code
## Now
Yaml lack of the information to track with original source code.
## Expected
Add source information to yaml files as following:
``` yaml
### YamlMime:Python
items:
- uid: azure.mgmt.devtestlabs.DevTestLabsClient
  source:
    remote:
      path: src/azure/DevTestLabsClient.cs
      branch: master
      repo: https://github.com/Microsoft/sdk
    id: DevTestLabsClient
    path: src/azure/DevTestLabsClient.cs
    startLine: 12
```

# Other missing fields in items
## inheritance
In .NET, it's a list tracing to `System.Object` like:
``` yaml
inheritance:
- System.Object
- VBTestClass1.BaseClass1
```
In python, considering multiple inheritance, it's also a list of all it's base classes:
``` yaml
inheritance:
- base1
- base2
...
```
## langs
``` yaml
uid: uid 
langs:
- python
```
## name
Before:
``` yaml
uid: uid 
name: cntk.io.MinibatchData.num_samples
```
Expected:
``` yaml
uid: uid 
fullName: cntk.io.MinibatchData.num_samples
name: num_samples
```
