**api-version**: can be any of the values shown in the table
**Kind**: can be any of the values shown in the table

![6ef8194b6bc4213928191120f0acf633.png](../_resources/6ef8194b6bc4213928191120f0acf633.png)

**Metadata**: Under metadata you can only provide name or lables or anything else that k8s expects. You cannot add anything of your own. But under lables, you can add anythnig you want.

**Spec:** Depends on the type of object we are creating. Here since the kind is set to POD, the spec section would contain containers as a list/array
![39ed2709a646402d29d6a57451b3e0cb.png](../_resources/39ed2709a646402d29d6a57451b3e0cb.png)