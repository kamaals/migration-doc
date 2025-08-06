API Development Plan
====================

Phase 1: Foundation and Data Migration
-------------------------------------

This phase focuses on establishing the foundational APIs for the core entities and migrating existing data into the system. Some tasks are complete, while others remain pending.

### Completed Tasks

- **Schema Review for Core Entities**: Reviewed the schema for the following core entities, with brief descriptions for context:
  - **organization**:
    - **Brief**: Previously referred to as "client" in Power Apps and MS SharePoint, this entity represents businesses or entities in the Postgres database, now standardized as `organization`.
    - **Schema**:
      ```json
      {
        "name": {
          "type": "string",
          "required": true,
          "description": "The official name of the organization, serving as the primary identifier."
        },
        "businessType": {
          "type": "string",
          "required": false,
          "description": "The type or category of the business (e.g., retail, manufacturing) for classification."
        },
        "description": {
          "type": "string",
          "required": false,
          "description": "A brief overview of the organization, providing context or additional details."
        },
        "isIntegrated": {
          "type": "boolean",
          "required": false,
          "description": "Indicates whether the organization is integrated with external systems or APIs."
        },
        "logo": {
          "type": "Array<string>",
          "required": false,
          "description": "A list of URLs or file paths pointing to the organization’s logo(s) for branding."
        },
        "merchantCode": {
          "type": "string",
          "required": false,
          "description": "A unique code assigned to the organization for merchant-related transactions or identification."
        },
        "metadata": {
          "type": "JSON",
          "required": false,
          "description": "Additional structured data for custom attributes or configuration specific to the organization."
        },
        "slug": {
          "type": "string",
          "required": false,
          "description": "A unique, auto-generated identifier derived from Hono middleware, not accepted from API requests, and inserted into the database."
        }
      }
      ```
  - **customer**:
    - **Brief**: Represents the end user who orders products through an organization’s platform (e.g., Shopify, WooCommerce, or custom websites). Customer data is stored in SharePoint with each `order`. Mr. Aminur and Mr. Mohsin will extract customer data, clean phone numbers to include proper country codes starting with a `+` symbol, and format it to match the API’s requirements for migration.
    - **Schema**:
      ```json
      {
        "name": {
          "type": "string",
          "required": false,
          "description": "The name of the customer, used for identification or display purposes."
        },
        "description": {
          "type": "string",
          "required": false,
          "description": "A brief overview of the customer, providing additional context or details."
        },
        "email": {
          "type": "string",
          "required": false,
          "description": "The customer’s email address. If provided, it must follow a valid email format; otherwise, a 400 Invalid response is returned."
        },
        "metadata": {
          "type": "JSON",
          "required": false,
          "description": "Additional structured data for custom attributes or configuration specific to the customer."
        },
        "organizationId": {
          "type": "uuid",
          "required": true,
          "description": "The UUID of the associated organization. If invalid or non-existent, an error is thrown."
        },
        "phone": {
          "type": "string",
          "required": false,
          "description": "The customer’s phone number. If it matches an existing phone number, the API returns the existing customer record. If invalid or empty, the API returns the 'no-user' (anonymous user) record without creating a new row. If valid and unique, a new customer record is created."
        },
        "addressLine1": {
          "type": "string",
          "required": false,
          "description": "The first line of the customer’s address, typically including street details."
        },
        "addressLine2": {
          "type": "string",
          "required": false,
          "description": "The second line of the customer’s address, for additional details like apartment or suite number."
        },
        "city": {
          "type": "string",
          "required": false,
          "description": "The city of the customer’s address."
        },
        "country": {
          "type": "string",
          "required": false,
          "description": "The country of the customer’s address."
        },
        "location": {
          "type": "latLng",
          "format": "{x: number, y: number}",
          "required": false,
          "description": "The geographic coordinates of the customer’s location, represented as latitude (x) and longitude (y)."
        }
      }
      ```
  - **product**:
    - **Brief**: Represents items available for order, initially extracted from SharePoint. In the future, product data will be sent from Power Apps and/or Power Automate (Flow) to the API for creation or updates.
    - **Schema**:
      ```json
      {
        "organizationId": {
          "type": "uuid",
          "required": true,
          "description": "The UUID of the associated organization. If invalid or non-existent, an error is thrown."
        },
        "barcode": {
          "type": "string",
          "required": true,
          "description": "A unique barcode identifier for the product, used for tracking or scanning."
        },
        "category": {
          "type": "string",
          "required": false,
          "description": "The category or type of the product (e.g., electronics, clothing) for classification."
        },
        "dimensions": {
          "type": "JSON",
          "required": false,
          "description": "Structured data describing the product’s dimensions (e.g., height, width, depth)."
        },
        "name": {
          "type": "string",
          "required": true,
          "description": "The official name of the product, used for identification or display."
        },
        "sku": {
          "type": "string",
          "required": true,
          "description": "The stock-keeping unit code for the product, used for inventory management."
        },
        "slug": {
          "type": "string",
          "required": false,
          "description": "A unique, auto-generated identifier derived from Hono middleware, not accepted from API requests, and inserted into the database."
        },
        "description": {
          "type": "string",
          "required": false,
          "description": "A brief overview of the product, providing additional context or details."
        },
        "hasExpiry": {
          "type": "boolean",
          "required": false,
          "description": "Indicates whether the product has an expiration date."
        },
        "isActive": {
          "type": "boolean",
          "required": false,
          "description": "Indicates whether the product is currently active or available."
        },
        "metadata": {
          "type": "JSON",
          "required": false,
          "description": "Additional structured data for custom attributes or configuration specific to the product."
        }
      }
      ```
  - **service**:
    - **Brief**: Represents services offered by an organization, initially loaded from SharePoint or added via Power Apps and/or Power Automate (Flow) to the API.
    - **Schema**:
      ```json
      {
        "name": {
          "type": "string",
          "required": true,
          "description": "The official name of the service, used for identification or display purposes."
        },
        "code": {
          "type": "string",
          "required": false,
          "description": "An internal code or identifier for the service, used for tracking or reference."
        },
        "description": {
          "type": "string",
          "required": false,
          "description": "A brief overview of the service, providing additional context or details."
        },
        "externalCode": {
          "type": "string",
          "required": false,
          "description": "An external code or identifier for the service, used for integration with external systems."
        },
        "slug": {
          "type": "string",
          "required": false,
          "description": "A unique, auto-generated identifier derived from Hono middleware, not accepted from API requests, and inserted into the database."
        },
        "parentId": {
          "type": "uuid",
          "required": false,
          "default": null,
          "description": "The UUID of the parent service’s slug.id, establishing a parent-child relationship. Defaults to NULL if no parent is specified."
        }
      }
      ```
  - **warehouse**:
    - **Brief**: Represents physical storage facilities for products, with data initially extracted from SharePoint. Includes address and location details to support inventory management.
    - **Schema**:
      ```json
      {
        "name": {
          "type": "string",
          "required": true,
          "description": "The official name of the warehouse, used for identification or display purposes."
        },
        "slug": {
          "type": "string",
          "required": false,
          "description": "A unique, auto-generated identifier derived from Hono middleware, not accepted from API requests, and inserted into the database."
        },
        "description": {
          "type": "string",
          "required": false,
          "description": "A brief overview of the warehouse, providing additional context or details."
        },
        "address": {
          "type": "string",
          "required": false,
          "description": "The street address of the warehouse, typically including street details."
        },
        "city": {
          "type": "string",
          "required": false,
          "description": "The city of the warehouse’s address."
        },
        "country": {
          "type": "string",
          "required": false,
          "description": "The country of the warehouse’s address."
        },
        "location": {
          "type": "latLng",
          "format": "{x: number, y: number}",
          "required": false,
          "description": "The geographic coordinates of the warehouse’s location, represented as latitude (x) and longitude (y)."
        },
        "metaData": {
          "type": "JSON",
          "required": false,
          "description": "Additional structured data for custom attributes or configuration specific to the warehouse."
        }
      }
      ```
  - **slot**:
    - **Brief**: Represents the organizational structure within a warehouse (e.g., zones, bins, racks), mapping the layout for inventory storage. Bin addresses are generated automatically upon creation.
    - **Schema**:
      ```json
      {
        "name": {
          "type": "string",
          "required": true,
          "description": "The official name of the slot, used for identification or display purposes."
        },
        "slug": {
          "type": "string",
          "required": false,
          "description": "A unique, auto-generated identifier derived from Hono middleware, not accepted from API requests, and inserted into the database."
        },
        "type": {
          "type": "enum",
          "values": ["floor", "zone", "aisle", "rack", "shelf", "bin", "container", "other"],
          "required": true,
          "description": "The type of slot, indicating its role in the warehouse structure (e.g., floor, zone, aisle)."
        },
        "binType": {
          "type": "enum",
          "values": ["Saleable", "Non-Saleable", "Damaged", "Staging", "Trolly-Bin"],
          "required": false,
          "description": "The type of bin within the slot, indicating its purpose (e.g., Saleable, Damaged). Applies primarily to slots of type 'bin'."
        },
        "parentId": {
          "type": "uuid",
          "required": false,
          "default": null,
          "description": "The UUID of the parent slot’s slug.id, establishing a parent-child relationship within the warehouse structure. Defaults to NULL if no parent is specified."
        },
        "metadata": {
          "type": "JSON",
          "required": false,
          "description": "Additional structured data for custom attributes or configuration specific to the slot."
        },
        "binSize": {
          "type": "JSON",
          "required": false,
          "description": "Structured data describing the size or capacity of the bin (e.g., volume, dimensions). Applies primarily to slots of type 'bin'."
        }
      }
      ```
- **API Development**: Created API endpoints for each core entity (`organization`, `customer`, `product`, `service`, `warehouse`, `slot`).
- **Documentation Creation**: Prepared initial documentation for the APIs.
- **Staging Deployment**: Deployed a staging version of the APIs for testing.

### Pending Tasks

- **Data Preparation**:
  - Mr. Aminur and Mr. Mohsin will prepare the SharePoint data and test the API. Once they confirm the API and its responses are satisfactory, they will proceed with migration using the prepared data.
    - **Deadline**: Pending (Mr. Aminur and Mr. Mohsin will update).
  - Ensure data aligns with the API schema for `organization`, `customer`, `product`, `service`, `warehouse`, and `slot`.
    - For `organization`, map fields previously labeled as "client" in SharePoint, exclude `slug` from mappings as it is auto-generated.
    - For `customer`, map fields like `name`, `email`, `phone`, `organizationId`, `addressLine1`, `addressLine2`, `city`, `country`, and `location` from SharePoint `order` data. Mr. Aminur and Mr. Mohsin will clean phone numbers to include proper country codes starting with `+` and format data to match API requirements. Ensure `organizationId` matches valid UUIDs in the `organization` table.
    - For `product`, map fields like `organizationId`, `barcode`, `category`, `dimensions`, `name`, `sku`, `description`, `hasExpiry`, `isActive`, and `metadata` from SharePoint, ensuring `organizationId` matches valid UUIDs and `barcode` and `sku` are unique.
    - For `service`, map fields like `name`, `code`, `description`, `externalCode`, and `parentId` from SharePoint, ensuring `name` is populated and `parentId` references valid `service` `slug.id` values or is `NULL`.
    - For `warehouse`, map fields like `name`, `description`, `address`, `city`, `country`, `location`, and `metaData` from SharePoint, ensuring `name` is populated, `location` follows the `{x: number, y: number}` format if provided, and excluding `slug` as it is auto-generated.
    - For `slot`, map fields like `name`, `type`, `binType`, `parentId`, `metadata`, and `binSize` from SharePoint, ensuring `name` and `type` are populated, `type` is a valid enum value, `binType` is a valid enum value if provided, `parentId` references valid `slot` `slug.id` values or is `NULL`, and excluding `slug` as it is auto-generated.
  - Validate `customer` data:
    - Ensure `email` fields, if provided, are in a valid format.
    - Ensure `phone` numbers are cleaned to include country codes starting with `+` and identify existing, invalid, or anonymous user cases per the specified logic.
    - Validate `location` fields, if provided, to ensure they follow the `{x: number, y: number}` format.
  - Validate `product` data:
    - Ensure required fields (`barcode`, `name`, `sku`, `organizationId`) are populated.
    - Validate `dimensions` as JSON and `organizationId` against existing `organization` records.
    - Exclude `slug` from mappings as it is auto-generated.
  - Validate `service` data:
    - Ensure `name` is populated.
    - Validate `parentId`, if provided, against existing `service` `slug.id` values; set to `NULL` if not provided.
    - Exclude `slug` from mappings as it is auto-generated.
  - Validate `warehouse` data:
    - Ensure `name` is populated.
    - Validate `location` fields, if provided, to ensure they follow the `{x: number, y: number}` format.
    - Transform `metaData` into JSON where applicable.
    - Exclude `slug` from mappings as it is auto-generated.
  - Validate `slot` data:
    - Ensure `name` and `type` are populated, with `type` being a valid enum value (`floor`, `zone`, `aisle`, `rack`, `shelf`, `bin`, `container`, `other`).
    - Validate `binType`, if provided, as a valid enum value (`Saleable`, `Non-Saleable`, `Damaged`, `Staging`, `Trolly-Bin`).
    - Validate `parentId`, if provided, against existing `slot` `slug.id` values; set to `NULL` if not provided.
    - Transform `metadata` and `binSize` into JSON where applicable.
    - Exclude `slug` from mappings as it is auto-generated.
  - Transform data into JSON for `metadata` (for `organization`, `customer`, `product`, `warehouse`, `slot`), `dimensions` (for `product`), and `binSize` (for `slot`) where applicable.
  - Plan for handling large data volumes to avoid performance bottlenecks during migration.
- **Data Migration**:
  - Migrate prepared data from MS SharePoint to the Postgres DB using the APIs, following confirmation from Mr. Aminur and Mr. Mohsin on API testing.
    - For `organization`, ensure required fields like `name` are populated, optional fields like `metadata` or `logo` are correctly formatted, and `slug` is auto-generated by Hono middleware.
    - For `customer`:
      - Validate `organizationId` against existing `organization` records.
      - Handle `phone` logic: return existing customer for duplicate phone numbers, return the 'no-user' record for invalid/empty phone numbers, and create new records for valid, unique phone numbers with proper country codes.
      - Validate `email` format, rejecting invalid entries with a 400 response.
      - Ensure `location` fields, if provided, are correctly formatted as `{x: number, y: number}`.
      - Store `addressLine1`, `addressLine2`, `city`, and `country` as provided.
    - For `product`:
      - Validate `organizationId` against existing `organization` records.
      - Ensure `barcode`, `name`, and `sku` are populated and unique.
      - Store `dimensions` as JSON and `slug` auto-generated by Hono middleware.
      - Store `category`, `description`, `hasExpiry`, `isActive`, and `metadata` as provided.
    - For `service`:
      - Ensure `name` is populated.
      - Validate `parentId`, if provided, against existing `service` `slug.id` values; set to `NULL` if not provided.
      - Store `code`, `description`, and `externalCode` as provided.
      - Ensure `slug` is auto-generated by Hono middleware.
    - For `warehouse`:
      - Ensure `name` is populated.
      - Store `description`, `address`, `city`, `country`, and `metaData` as provided.
      - Ensure `location` fields, if provided, are correctly formatted as `{x: number, y: number}`.
      - Ensure `slug` is auto-generated by Hono middleware.
    - For `slot`:
      - Ensure `name` and `type` are populated, with `type` being a valid enum value.
      - Validate `binType`, if provided, as a valid enum value.
      - Validate `parentId`, if provided, against existing `slot` `slug.id` values; set to `NULL` if not provided.
      - Store `metadata` and `binSize` as JSON where applicable.
      - Ensure `slug` is auto-generated by Hono middleware.
      - Ensure bin addresses are generated automatically during creation.
- **Verification**:
  - Verify successful migration, including:
    - Data integrity checks (e.g., all `organization` records have a valid `name` and unique `slug`, all `customer` records have a valid `organizationId`, all `product` records have valid `barcode`, `name`, `sku`, and `organizationId`, all `service` records have a valid `name` and valid or `NULL` `parentId`, all `warehouse` records have a valid `name` and unique `slug`, all `slot` records have a valid `name`, `type`, and unique `slug`).
    - Validate `customer` phone logic: confirm correct handling of existing, invalid, or anonymous user cases, with phone numbers formatted with `+` country codes.
    - Verify `customer` `email` format and `location` structure.
    - Verify `product` `slug` uniqueness and `dimensions` JSON structure.
    - Verify `service` `slug` uniqueness and `parentId` relationships.
    - Verify `warehouse` `slug` uniqueness, `location` structure, and `metaData` JSON structure.
    - Verify `slot` `slug` uniqueness, `type` and `binType` enum values, `parentId` relationships, `metadata` and `binSize` JSON structure, and automatic bin address generation.
    - Confirm optional fields (e.g., `organization.logo`, `customer.metadata`, `product.metadata`, `service.description`, `warehouse.metaData`, `slot.metadata`, `slot.binSize`) are correctly formatted.
    - Verify row counts and consistency between SharePoint and Postgres.

### Deliverables

- Successful migration of data from MS SharePoint to the Postgres DB, with `organization`, `customer`, `product`, `service`, `warehouse`, and `slot` data adhering to their schemas, including `slug` auto-generation, `customer.phone` logic with cleaned phone numbers, `product` required fields, `service` and `slot` parent-child relationships, `warehouse` address and location fields, and `slot` enum fields and bin address generation, following API testing by Mr. Aminur and Mr. Mohsin.
- A stable staging environment with fully functional core APIs.

Phase 2: Enhancement and Integration
-----------------------------------

This phase builds on Phase 1 by enhancing the APIs, adding new features, integrating the `order`, `items`, and `platform` entities, and establishing a many-to-many relationship between `organization` and `platform`. Feedback from testing will drive improvements.

### Tasks

- **Schema Addition for New Entities**:
  - **order**:
    - **Brief**: Represents customer orders placed through an organization’s platform, initially extracted from SharePoint. Contains critical details about the order, including customer information, payment, and shipping details, serving as the main entity for transactional data.
    - **Schema**:
      ```json
      {
        "order_number": {
          "type": "string",
          "required": true,
          "description": "A unique identifier for the order, used for tracking and reference across systems."
        },
        "organization_id": {
          "type": "uuid",
          "required": true,
          "description": "The UUID of the associated organization, linking the order to a specific business entity. Must reference a valid `organization` record."
        },
        "platform_id": {
          "type": "string",
          "required": false,
          "description": "The identifier of the platform (e.g., Shopify, WooCommerce) where the order was placed, used for integration and tracking the order’s source."
        },
        "customer_name": {
          "type": "string",
          "required": false,
          "description": "The name of the customer who placed the order, used for reference and display purposes."
        },
        "customer_email": {
          "type": "string",
          "required": false,
          "description": "The customer’s email address associated with the order. If provided, it must follow a valid email format; otherwise, a 400 Invalid response is returned."
        },
        "total_amount": {
          "type": "number",
          "required": true,
          "description": "The total monetary amount of the order, including taxes and shipping costs, used for financial tracking and reporting."
        },
        "status": {
          "type": "string",
          "required": true,
          "description": "The current status of the order (e.g., 'pending', 'processing', 'shipped', 'delivered', 'cancelled'), used to track the order lifecycle."
        },
        "shipping_address": {
          "type": "string",
          "required": false,
          "description": "The full shipping address for the order, including street details, city, and country, used for delivery logistics."
        },
        "billing_address": {
          "type": "string",
          "required": false,
          "description": "The billing address associated with the order, used for payment processing and verification."
        },
        "payment_method": {
          "type": "string",
          "required": false,
          "description": "The payment method used for the order (e.g., 'credit_card', 'paypal', 'cod'), used for financial reconciliation."
        },
        "notes": {
          "type": "string",
          "required": false,
          "description": "Additional comments or instructions related to the order, provided by the customer or organization for context or special handling."
        }
      }
      ```
  - **items**:
    - **Brief**: Represents individual items within an order, linking products to specific orders with quantity and pricing details. Data is initially extracted from SharePoint and associated with the `order` entity.
    - **Schema**:
      ```json
      {
        "order_id": {
          "type": "uuid",
          "required": true,
          "description": "The UUID of the associated order, linking the item to a specific order. Must reference a valid `order` record."
        },
        "product_id": {
          "type": "uuid",
          "required": true,
          "description": "The UUID of the associated product, linking the item to a specific product. Must reference a valid `product` record."
        },
        "quantity": {
          "type": "integer",
          "required": true,
          "description": "The number of units of the product ordered, used for inventory and order fulfillment calculations."
        },
        "unit_price": {
          "type": "numeric",
          "required": true,
          "description": "The price per unit of the product at the time of the order, used for financial calculations."
        },
        "total_price": {
          "type": "numeric",
          "required": true,
          "description": "The total price for the item (quantity * unit_price), used for order summary and financial reporting."
        }
      }
      ```
  - **platform**:
    - **Brief**: Represents the platforms (e.g., Shopify, WooCommerce) used by organizations to process orders, with a many-to-many relationship with the `organization` table. Data is initially extracted from SharePoint or added via Power Apps/Flow.
    - **Schema**:
      ```json
      {
        "name": {
          "type": "string",
          "required": true,
          "description": "The name of the platform (e.g., Shopify, WooCommerce), used for identification and display purposes."
        },
        "description": {
          "type": "string",
          "required": false,
          "description": "A brief overview of the platform, providing context or additional details about its functionality or purpose."
        },
        "url": {
          "type": "string",
          "required": false,
          "description": "The URL of the platform’s website or API endpoint, used for integration or reference."
        },
        "api_key": {
          "type": "string",
          "required": false,
          "description": "The API key for authenticating with the platform’s API, used for secure integration with external systems."
        },
        "is_active": {
          "type": "boolean",
          "required": false,
          "default": true,
          "description": "Indicates whether the platform is currently active for use by the organization. Defaults to true."
        }
      }
      ```
    - **Relationship**: A many-to-many relationship between `organization` and `platform` will be implemented via a junction table (e.g., `organization_platform`), with fields `organization_id` (UUID referencing `organization`) and `platform_id` (UUID referencing `platform`).
    - **return**:
       - **Brief**: Represents return requests associated with orders, tracking the return process, quantities, and statuses for inventory and customer service purposes. Data is initially extracted from SharePoint and linked to `order` and `organization`.
       - **Brief**:
         ```json
          {
            "organization_id": {
              "type": "uuid",
              "required": true,
              "description": "The UUID of the associated organization, linking the return to a specific business entity. Must reference a valid `organization` record."
            },
            "order_id": {
              "type": "uuid",
              "required": true,
              "description": "The UUID of the associated order, linking the return to a specific order. Must reference a valid `order` record."
            },
            "return_number": {
              "type": "string",
              "required": true,
              "description": "A unique identifier for the return request, used for tracking and reference across systems."
            },
            "return_type": {
              "type": "enum",
              "values": ["full_refunded", "partial_refunded", "exchange", "replacement", "other"], // Placeholder: please provide specific enum values
              "required": true,
              "description": "The type of return request (e.g., full refund, partial refund, exchange), used to categorize the return process."
            },
            "document_status": {
              "type": "enum",
              "values": ["pending", "approved", "rejected", "processed", "cancelled"], // Placeholder: please provide specific enum values
              "required": true,
              "description": "The status of the return documentation, indicating the progress of return approval or processing."
            },
            "awb": {
              "type": "string",
              "required": false,
              "description": "The Air Waybill (AWB) number for the return shipment, used for tracking logistics."
            },
            "total_quantity": {
              "type": "integer",
              "required": false,
              "default": 0,
              "description": "The total number of items included in the return request, used for inventory tracking."
            },
            "putaway_quantity": {
              "type": "integer",
              "required": false,
              "default": 0,
              "description": "The number of returned items processed and placed back into inventory, used for warehouse management."
            },
            "putaway_status_id": {
              "type": "uuid",
              "required": false,
              "description": "The UUID of the associated putaway status record, linking to a specific inventory status (e.g., in a putaway status table). Must reference a valid record if provided."
            },
            "processed_by": {
              "type": "uuid",
              "required": false,
              "description": "The UUID of the user or system processing the return, used for accountability and tracking."
            },
            "notes": {
              "type": "string",
              "required": false,
              "description": "Additional comments or instructions related to the return, provided by the customer or organization for context or special handling."
            },
            "status_type": {
              "type": "enum",
              "values": ["initiated", "in_progress", "completed", "cancelled"], // Placeholder: please provide specific enum values
              "required": true,
              "description": "The overall status of the return process, indicating its current stage in the return lifecycle."
            }
          }
         ```
       - **Note**: The `return_type`, `document_status`, and `status_type` fields are defined as enums with placeholder values. Please provide the specific enum values for accurate implementation.





- **Feedback Review**:
  - Collect feedback from staging environment testing, including issues with `organization` (e.g., `slug` generation), `customer` (e.g., `phone` logic, `email` validation, `location` format), `product` (e.g., `barcode` or `sku` uniqueness, `dimensions` parsing), `service` (e.g., `slug` generation, `parentId` relationships), `warehouse` (e.g., `slug` generation, `location` format, `metaData` parsing), `slot` (e.g., `slug` generation, `type` or `binType` validation, `parentId` relationships, bin address generation), `order` (e.g., `order_number` uniqueness, `organization_id` validation, `customer_email` format, `status` handling), `items` (e.g., `order_id` and `product_id` validation, `quantity` and pricing accuracy), and `platform` (e.g., `api_key` security, `is_active` handling, many-to-many relationship with `organization`).
- **API Refinement**:
  - Implement changes or bug fixes, such as:
    - Ensuring `slug` uniqueness for `organization`, `product`, `service`, `warehouse`, and `slot`.
    - Correct handling of `customer` `phone` logic (e.g., consistent responses for existing or anonymous users with proper `+` country code formatting).
    - Robust `customer` and `warehouse` `location` format validation and `email` format validation for `customer` and `order.customer_email`.
    - Ensuring `product` `barcode` and `sku` uniqueness and `dimensions` JSON validation.
    - Validating `service` and `slot` `parentId` relationships and ensuring no circular dependencies.
    - Validating `warehouse` `metaData` and `slot` `metadata` and `binSize` JSON structure.
    - Ensuring `slot` `type` and `binType` adhere to their respective enum values and bin addresses are generated correctly.
    - Ensuring `order` fields like `order_number` are unique, `organization_id` references valid `organization` records, `customer_email` follows valid email format, and `status` is properly managed.
    - Validating `items` fields like `order_id` and `product_id` against existing `order` and `product` records, and ensuring `quantity`, `unit_price`, and `total_price` are consistent (e.g., `total_price` = `quantity` * `unit_price`).
    - Ensuring `platform` fields like `name` are populated, `api_key` is securely handled, and the many-to-many `organization_platform` relationship is correctly implemented.
- **Feature Expansion**:
  - Develop additional API endpoints for all entities, including:
    - Querying `organization` by `slug`, `customer` by `phone` or `email`, `product` by `barcode`, `sku`, or `category`, `service` by `code`, `externalCode`, or `parentId`, `warehouse` by `name`, `slug`, or `city`, `slot` by `name`, `type`, `binType`, or `parentId`, `order` by `order_number`, `organization_id`, `platform_id`, or `status`, `items` by `order_id` or `product_id`, and `platform` by `name` or `is_active`.
    - Support for bulk operations (e.g., batch updates for `customer`, `product`, `service`, `warehouse`, `slot`, `order`, `items`, or `platform` records, excluding `slug` modifications).
    - Geospatial query support for `customer.location` and `warehouse.location` (e.g., find customers or warehouses within a radius).
    - Filtering for `product` by `hasExpiry` or `isActive`, for `service` by `parentId`, for `slot` by `type`, `binType`, or `parentId`, for `order` by `status` or `payment_method`, for `items` by `quantity` or `total_price`, and for `platform` by `is_active`.
    - Support for querying the many-to-many relationship between `organization` and `platform` (e.g., retrieve all platforms for an organization or all organizations for a platform).
  - Add support for Power Apps/Flow integration for `product`, `service`, `order`, `items`, and `platform` data updates.
- **Security Enhancements**:
  - Ensure secure handling of sensitive fields like `organization.merchantCode`, `organization.metadata`, `customer.email`, `customer.phone`, `customer.location`, `product.barcode`, `product.sku`, `product.metadata`, `service.externalCode`, `service.metadata`, `warehouse.location`, `warehouse.metaData`, `slot.metadata`, `slot.binSize`, `order.customer_email`, `order.shipping_address`, `order.billing_address`, `order.payment_method`, `items.unit_price`, `items.total_price`, and `platform.api_key` (e.g., encryption, access controls).
  - Prevent manual updates to `organization.slug`, `product.slug`, `service.slug`, `warehouse.slug`, and `slot.slug` via API.
  - Implement or enhance authentication (e.g., OAuth, JWT) for API access, including for `order`, `items`, and `platform` endpoints.
- **System Integration**:
  - Integrate APIs with external systems, such as Power Apps/Flow for `product`, `service`, `order`, `items`, and `platform` updates, using fields like `organization.isIntegrated`, `organization.slug`, `customer.organizationId`, `product.organizationId`, `product.barcode`, `service.externalCode`, `service.parentId`, `warehouse.slug`, `warehouse.location`, `slot.type`, `slot.binType`, `slot.parentId`, `order.order_number`, `order.organization_id`, `order.platform_id`, `items.order_id`, `items.product_id`, `platform.api_key`, and `platform.url` for identification, and potentially `customer.location` and `warehouse.location` for geospatial integrations.
  - Implement the many-to-many relationship between `organization` and `platform` via a junction table (`organization_platform`) for seamless integration.
- **Documentation Update**:
  - Update API documentation to reflect new endpoints or changes, including examples for `organization`, `customer`, `product`, `service`, `warehouse`, `slot`, `order`, `items`, and `platform` endpoints, with notes on `slug` auto-generation, `customer.phone` logic with country code formatting, `product` required fields, `service` and `slot` parent-child relationships, `warehouse` address and location fields, `slot` enum fields and bin address generation, `order` fields like `order_number` and `status`, `items` pricing consistency, `platform` fields like `api_key`, and the `organization_platform` many-to-many relationship.
- **Testing**:
  - Conduct integration testing, ensuring `organization`, `customer`, `product`, `service`, `warehouse`, `slot`, `order`, `items`, and `platform` data (including `slug`, `phone`, `location`, `barcode`, `sku`, `parentId`, `type`, `binType`, `metadata`, `binSize`, `order_number`, `unit_price`, `total_price`, `platform.api_key`, and `organization_platform` relationships) integrate correctly with other entities, Power Apps/Flow, or external systems.
- **Staging Update**:
  - Deploy an updated staging version for further testing, including `order`, `items`, and `platform` endpoints.

### Deliverables

- Enhanced APIs with additional features and integrations, including support for `customer` and `warehouse` address and `location` fields, `product` fields like `barcode` and `sku`, `service` and `slot` parent-child relationships, `warehouse` `metaData`, `slot` enum fields and `binSize`, `order` fields like `order_number` and `status`, `items` fields like `quantity` and `total_price`, `platform` fields like `api_key` and `is_active`, and the `organization_platform` many-to-many relationship, with Power Apps/Flow integration for `product`, `service`, `order`, `items`, and `platform`.
- Updated API documentation, including examples for `organization`, `customer`, `product`, `service`, `warehouse`, `slot`, `order`, `items`, and `platform` endpoints, `slug` behavior, `customer.phone` logic, `service.parentId` and `slot.parentId` relationships, `slot` enum fields, bin address generation, `order` fields, `items` pricing, `platform` fields, and `organization_platform` relationships.
- A stable staging environment ready for final testing.

Phase 3: Finalization and Deployment
-----------------------------------

This phase finalizes the migration, redirects all operations data storage flows and Power Apps to use the API, and prepares the APIs for production, ensuring they are robust, secure, and fully documented. The expected deadline is September 19, 2025, after which all Power Apps and Flows must use the API in production.

### Tasks

- **Final Testing**:
  - Conduct comprehensive testing to finalize migration and ensure API readiness, including:
    - Performance testing (e.g., handling large `organization`, `customer`, `product`, `service`, `warehouse`, `slot`, `order`, `items`, and `platform` datasets with unique `slug`, `phone`, `location`, `barcode`, `sku`, `parentId`, `type`, `binType`, `metadata`, `binSize`, `order_number`, and `platform.api_key`).
    - Security testing (e.g., ensuring `merchantCode`, `metadata`, `email`, `phone`, `location`, `barcode`, `sku`, `externalCode`, `parentId`, `description`, `metaData`, `type`, `binType`, `binSize`, `customer_email`, `shipping_address`, `billing_address`, `payment_method`, `unit_price`, `total_price`, and `api_key` are protected).
    - Integration testing with Power Apps and Flows to ensure all operations data storage (e.g., `product`, `service`, `order`, `items`, `platform`) is correctly routed through the API, with proper handling of `organization_platform` relationships and `items` pricing consistency.
- **Issue Resolution**:
  - Address remaining issues, such as errors in `slug` generation, `customer.phone` logic, `customer.email` or `order.customer_email` validation, `product` `barcode`, `sku`, or `dimensions` parsing, `service` or `slot` `parentId` relationships, `warehouse` `location` or `metaData` parsing, `slot` `type`, `binType`, `binSize`, or bin address generation, `order` `order_number` uniqueness or `status` handling, `items` `order_id` or `product_id` validation, `platform` `api_key` security, `organization_platform` relationship handling, or Power Apps/Flow integration failures.
- **Documentation Finalization**:
  - Finalize API documentation, ensuring clear instructions for `organization` (including `slug`), `customer` (including `phone`, `email`, `location`), `product` (including `barcode`, `sku`, `slug`, `dimensions`), `service` (including `slug`, `parentId`), `warehouse` (including `slug`, `location`, `metaData`), `slot` (including `slug`, `type`, `binType`, `parentId`, `binSize`, bin address generation), `order` (including `order_number`, `organization_id`, `status`), `items` (including `order_id`, `product_id`, `total_price`), and `platform` (including `api_key`, `is_active`, `organization_platform` relationships).
  - Include specific guidance for Power Apps and Flow developers on integrating with the API for data storage operations by September 19, 2025.
- **Production Preparation**:
  - Set up production environments, ensuring the Postgres DB schema matches the `organization`, `customer`, `product`, `service`, `warehouse`, `slot`, `order`, `items`, and `platform` schemas, including all required and optional fields and the `organization_platform` junction table.
  - Coordinate with the Power Apps and Flow teams to redirect all operations data storage (e.g., `product`, `service`, `order`, `items`, `platform`) to the API, ensuring all flows are updated to use API endpoints by the deadline of September 19, 2025.
  - Verify that all migrated data from SharePoint is complete and accurate, with no pending migrations for `organization`, `customer`, `product`, `service`, `warehouse`, or `slot`.
- **Deployment**:
  - Deploy the APIs to production by September 19, 2025, ensuring `organization`, `customer`, `product`, `service`, `warehouse`, `slot`, `order`, `items`, and `platform` data, including `slug`, `phone`, `location`, `barcode`, `sku`, `parentId`, `type`, `binType`, `binSize`, `order_number`, `unit_price`, `total_price`, and `api_key`, are accessible and functional.
  - Ensure all Power Apps and Flows are fully transitioned to use the API for data storage operations in production starting September 19, 2025.
- **Post-Deployment**:
  - Monitor production for issues, such as errors with `organization`, `product`, `service`, `warehouse`, or `slot` `slug` conflicts, `customer.phone` logic, `service` or `slot` `parentId` queries, `slot` `type`, `binType`, or bin address generation, `order` `order_number` conflicts or `status` updates, `items` pricing inconsistencies, `platform` `api_key` security, `organization_platform` relationship issues, or Power Apps/Flow integration failures.
- **Maintenance Planning**:
  - Plan for ongoing maintenance, including schema updates for `organization`, `customer`, `product`, `service`, `warehouse`, `slot`, `order`, `items`, or `platform` if new fields are needed, and support for Power Apps/Flow integrations post-September 19, 2025.

### Deliverables

- Production-ready APIs, fully tested and deployed by September 19, 2025, with all Power Apps and Flows using the API for data storage operations in production.
- Comprehensive, finalized API documentation, including `organization`, `customer`, `product`, `service`, `warehouse`, `slot`, `order`, `items`, and `platform` schemas with `slug`, `phone`, `location`, `barcode`, `sku`, `parentId`, `type`, `binType`, `binSize`, `order_number`, `unit_price`, `total_price`, `api_key`, and `organization_platform` relationship details, and specific guidance for Power Apps/Flow integration.
- A monitoring and maintenance plan for post-deployment support, ensuring continued functionality of API-based operations.

Notes
-----

- **Phase 3 Update**: Added tasks to finalize migration, redirect all Power Apps and Flows to use the API for data storage operations, and set a deadline of September 19, 2025, for full production use. These changes are integrated into **Final Testing**, **Production Preparation**, and **Deployment** tasks to ensure a seamless transition.
- **Migration Considerations**:
  - For `organization`, map SharePoint "client" data to `organization` fields, exclude `slug`, and validate uniqueness.
  - For `customer`, ensure `organizationId` references valid `organization` records, validate `email` and `location` formats, implement `phone` logic with cleaned `+` country codes, and rely on Mr. Aminur and Mr. Mohsin for data extraction, formatting, and API testing from SharePoint `order` data.
  - For `product`, ensure `organizationId` references valid `organization` records, validate `barcode`, `name`, and `sku` as required and unique, exclude `slug`, validate `dimensions` as JSON, and prepare for Power Apps/Flow integration.
  - For `service`, ensure `name` is populated, validate `parentId` against existing `service` `slug.id` values or set to `NULL`, exclude `slug`, store `code`, `description`, and `externalCode` as provided, and prepare for Power Apps/Flow integration.
  - For `warehouse`, ensure `name` is populated, validate `location` format, transform `metaData` to JSON, exclude `slug`, and store `address`, `city`, `country`, and `description` as provided.
  - For `slot`, ensure `name` and `type` are populated, validate `type` and `binType` as valid enum values, validate `parentId` against existing `slot` `slug.id` values or set to `NULL`, transform `metadata` and `binSize` to JSON, exclude `slug`, and ensure automatic bin address generation.
  - For `order`, ensure `order_number` is unique, `organization_id` references valid `organization` records, `customer_email` follows valid email format, and store `platform_id`, `customer_name`, `total_amount`, `status`, `shipping_address`, `billing_address`, `payment_method`, and `notes` as provided. Prepare for Power Apps/Flow integration.
  - For `items`, ensure `order_id` and `product_id` reference valid `order` and `product` records, validate `quantity`, `unit_price`, and `total_price` for consistency, and prepare for Power Apps/Flow integration.
  - For `platform`, ensure `name` is populated, store `description`, `url`, `api_key`, and `is_active` as provided, implement the `organization_platform` junction table for the many-to-many relationship, and prepare for Power Apps/Flow integration.
- **Data Preparation Update**: Phase 1 includes details that Mr. Aminur and Mr. Mohsin will prepare SharePoint data, test the API, and proceed with migration upon satisfaction with API responses, with a pending deadline to be updated by them.
- **Flexibility**: If you have specific requirements for Phases 2 or 3 (e.g., geospatial queries for `customer.location` or `warehouse.location`, inventory integrations for `product.barcode` or `sku`, hierarchical queries for `service.parentId` or `slot.parentId`, slot-specific integrations for `type` or `binType`, or additional details for Power Apps/Flow integration by September 19, 2025), let me know, and I can refine the plan further.
