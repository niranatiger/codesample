# how can I use dynamic to implement terraform module, one will assign body with openapi spec file, and another not?

To implement a Terraform module that supports dynamic behavior with an optional OpenAPI specification file, you can follow these steps:

Define module variables: Create a Terraform module and declare input variables to allow customization. For example, you can define a variable named openapi_spec_file to represent the OpenAPI specification file.

    variable "openapi_spec_file" {
      description = "Path to the OpenAPI specification file"
      type        = string
      default     = null
    }

Handle dynamic behavior: Within your module, you can use the dynamic block to conditionally include resources based on the presence of the OpenAPI specification file. You can use the fileexists function to check if the file exists.

    data "template_file" "api_gateway" {
      count    = var.openapi_spec_file != null && fileexists(var.openapi_spec_file) ? 1 : 0
      template = file("${path.module}/templates/api_gateway.tf")
      vars = {
        openapi_spec_file = var.openapi_spec_file
      }
    }

    resource "aws_api_gateway_rest_api" "api_gateway" {
      count = data.template_file.api_gateway[*].rendered != null ? 1 : 0

      # Rest of the resource configuration
    }
    
In this example, the dynamic block is used to conditionally include the aws_api_gateway_rest_api resource only if the OpenAPI specification file exists. The data.template_file.api_gateway data block generates the configuration for the resource by rendering a template (templates/api_gateway.tf) that uses the provided OpenAPI specification file.

Handle module output: If you have module outputs, make sure to handle the dynamic behavior in their definition as well.

    output "api_gateway_id" {
      value       = aws_api_gateway_rest_api.api_gateway[0].id
      description = "ID of the API Gateway"
      depends_on  = [aws_api_gateway_rest_api.api_gateway]
      count       = aws_api_gateway_rest_api.api_gateway[*].id != null ? 1 : 0
    }
    
Here, the output value is conditionally assigned based on the presence of the aws_api_gateway_rest_api resource.

By following these steps, you can create a Terraform module that allows you to optionally assign the OpenAPI specification file and conditionally include resources based on its availability.

# I use this way for the resouce aws_api_gateway_resouce, but hit the error "because aws_api_gateway_resouce.proxy_resouce has "count" set, its attributes must be accessed on specifict instances.", it says Missing resouce instance key

I apologize for the confusion. The error message you encountered suggests that when using the count argument with the aws_api_gateway_resource resource, you need to access the attributes of the resource using specific instance indices.

The count argument creates multiple instances of a resource, and each instance must be accessed individually. To resolve the error, you should modify your code to access the attributes of each instance using the instance indices.

Here's an example of how you can modify your code:


    resource "aws_api_gateway_resource" "proxy_resource" {
      count    = var.create_resource ? 1 : 0
      rest_api_id = aws_api_gateway_rest_api.example.id
      parent_id   = aws_api_gateway_rest_api.example.root_resource_id
      path_part   = var.path_part
    
      # Other resource configuration
    }

To access the attributes of the aws_api_gateway_resource instances, you need to specify the instance indices explicitly. For example, to access the path attribute of the first instance, you would use:


    aws_api_gateway_resource.proxy_resource[0].path
    
If you have multiple instances (e.g., count = 3), you can access the attributes of each instance using their respective indices:

    aws_api_gateway_resource.proxy_resource[0].path
    aws_api_gateway_resource.proxy_resource[1].path
    aws_api_gateway_resource.proxy_resource[2].path
    
Make sure to adjust your code accordingly, accessing the attributes of the specific instances you are interested in.
