## [CRD & CRs - excercise]:
----------
(created a `Shirt` CR resource)
kl get Shirt

        NAME         COLOR   SIZE
        blue-shirt   blue    M-39

kl describe Shirt blue-shirt

        Name:         blue-shirt
        Namespace:    stage-1-practicals
        Labels:       app=clothing-store
                    brand=allen-solly
                    category=casual
        Annotations:  created-by: Allen Solly Inc.
                    description: Premium blue casual shirt
        API Version:  stable.example.com/v1
        Kind:         Shirt
        Metadata:
        Creation Timestamp:  2025-06-11T23:00:32Z
        Generation:          1
        Resource Version:    95172
        UID:                 6cf31a22-47a7-4f1c-9fa0-945cd276fa07
        Spec:
        Color:  blue
        Size:   M-39
        Events:   <none>



        