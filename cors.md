
To support CORS (Cross-Origin Resource Sharing) in AWS API Gateway, you need to configure the API Gateway resources with the appropriate CORS headers. Here's how you can achieve that using Terraform:

Enable CORS for the API Gateway resource:

  resource "aws_api_gateway_rest_api" "api_gateway" {
    name        = "MyAPIGateway"
    description = "My API Gateway"

    endpoint_configuration {
      types = ["REGIONAL"]
    }

    # Other resource configurations
  }
Configure the OPTIONS method for the resource to handle pre-flight CORS requests:

  resource "aws_api_gateway_method" "options" {
    rest_api_id   = aws_api_gateway_rest_api.api_gateway.id
    resource_id   = aws_api_gateway_resource.resource.id
    http_method   = "OPTIONS"
    authorization = "NONE"

    request_parameters = {
      "method.request.header.Origin" = true
      "method.request.header.Access-Control-Allow-Headers" = true
      "method.request.header.Access-Control-Allow-Methods" = true
      "method.request.header.Access-Control-Allow-Origin"  = true
      "method.request.header.Access-Control-Allow-Credentials" = true
    }

    request_validator_id = aws_api_gateway_request_validator.validator.id
  }
Add a response to the OPTIONS method to include the necessary CORS headers:

  resource "aws_api_gateway_integration_response" "options" {
    rest_api_id     = aws_api_gateway_rest_api.api_gateway.id
    resource_id     = aws_api_gateway_resource.resource.id
    http_method     = aws_api_gateway_method.options.http_method
    status_code     = "200"
    selection_pattern = ""

    response_parameters = {
      "method.response.header.Access-Control-Allow-Headers" = "'Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token'"
      "method.response.header.Access-Control-Allow-Methods" = "'OPTIONS,GET,POST,PUT,DELETE'"
      "method.response.header.Access-Control-Allow-Origin"  = "'*'"
      "method.response.header.Access-Control-Allow-Credentials" = "'true'"
    }

    response_templates = {
      "application/json" = ""
    }
  }
Associate the OPTIONS method response with the appropriate method response:

  resource "aws_api_gateway_method_response" "options" {
    rest_api_id = aws_api_gateway_rest_api.api_gateway.id
    resource_id = aws_api_gateway_resource.resource.id
    http_method = aws_api_gateway_method.options.http_method
    status_code = aws_api_gateway_integration_response.options.status_code

    response_models = {
      "application/json" = "Empty"
    }
  }
Deploy the API Gateway to make the CORS changes effective:

  resource "aws_api_gateway_deployment" "deployment" {
    rest_api_id = aws_api_gateway_rest_api.api_gateway.id
    stage_name  = "prod"
  }
By following these steps and customizing the configurations as per your API Gateway resource setup, you can enable CORS for your AWS API Gateway using Terraform.
