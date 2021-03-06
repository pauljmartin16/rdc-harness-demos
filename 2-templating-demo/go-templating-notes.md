# Go Templating Demo
- work through some Go templating samples. Helm templating is based on Go
- yaml content taken from the excellent Pluralsight course at https://app.pluralsight.com/library/courses/kubernetes-packaging-applications-helm/table-of-contents
- Ingress yamls updated from /v1beta to /v1 format

# Pre-Requisites
```
Install Go - https://go.dev/doc/install
Git samples available locally in Go-Template/
** OLD - Download Samples - https://github.com/phcollignon/Go-Template.git **
```

# Prepare the Template Binary
- cd \src in the Go-Template source
```
go build

add Go-Template.exe to your environment PATH
```
# Walk through the examples
- tip - leave 000-markdown to the end
- suggest using the yaml examples. However works equally with Json & Yaml

- an example in the 01-values\ directory:
```
Go-Template -d email.json -t email.tpl

output is written to email.generated.txt
```

# Comparing to Harness
- Rolling deploy yamls act as templates
- RDC values directory provides substitution values
- Harness runs Helm template operation to build working yaml configs
