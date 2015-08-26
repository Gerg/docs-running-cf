---
title: Usage Events and Billing
---

## Usage Events
Usage events can be used by Cloud Foundry operators to construct billing information for apps and service instances.

### App Usage Events

App usage events provide information about when users create and delete apps. They also include information about the app to enable operators to set billing rates based on resource usage.

#### Endpoint

`GET /v2/app_usage_events`

#### Example Response
```
{
  "total_results": 2,
  "total_pages": 2,
  "prev_url": null,
  "next_url": "/v2/app_usage_events?after_guid=d8dbe8af-94a1-4970-88e8-6fac56c0b7db&order-direction=asc&page=2&results-per-page=1",
  "resources": [
    {
      "metadata": {
        "guid": "7a63d717-3316-402f-bf12-a2a63211d1b9",
        "url": "/v2/app_usage_events/7a63d717-3316-402f-bf12-a2a63211d1b9",
        "created_at": "2015-08-06T00:36:46Z"
      },
      "entity": {
        "state": "STARTED",
        "memory_in_mb_per_instance": 564,
        "instance_count": 1,
        "app_guid": "guid-ed165c78-56d3-40a7-aafe-93933fe65138",
        "app_name": "name-1812",
        "space_guid": "guid-f3fb7d5d-beb9-4e0f-a842-babe13173ebb",
        "space_name": "name-1813",
        "org_guid": "guid-e04c4cdb-e42f-47e7-9323-718b3890ee0f",
        "buildpack_guid": "guid-9d07e5c4-48ce-496e-aa53-b7535fc75caf",
        "buildpack_name": "name-1814",
        "package_state": "STAGED",
        "parent_app_guid": null,
        "parent_app_name": null,
        "process_type": "web"
      }
    }
  ]
}
```

#### How to Use

Valid states:
- STARTED
- STOPPED
- BUILDPACK_SET <--- Why, Do we care for billing?

### Service Usage Events

Service usage events provide information about when users create, update, and delete service instances.

#### Endpoint

`GET /v2/service_usage_events`

#### Example Response

```
{
  "total_results": 2,
  "total_pages": 2,
  "prev_url": null,
  "next_url": "/v2/service_usage_events?after_guid=e8c3e67c-39f6-4598-ae2e-716c74a2a791&order-direction=asc&page=2&results-per-page=1",
  "resources": [
    {
      "metadata": {
        "guid": "8727031a-9b37-464f-b9c2-b3b75c5a393d",
        "url": "/v2/service_usage_events/8727031a-9b37-464f-b9c2-b3b75c5a393d",
        "created_at": "2015-08-06T00:36:45Z"
      },
      "entity": {
        "state": "CREATED",
        "org_guid": "guid-b5a56971-5432-4829-870b-0cb2a5cddbeb",
        "space_guid": "guid-b1b2499c-f406-494f-bfd5-42308b734a3f",
        "space_name": "name-1689",
        "service_instance_guid": "guid-67f5738a-aac4-4f42-a437-41a6c6920978",
        "service_instance_name": "name-1690",
        "service_instance_type": "type-8",
        "service_plan_guid": "guid-28cd72d6-91b6-40d8-9ae4-6357768d4ab4",
        "service_plan_name": "name-1691",
        "service_guid": "guid-251a9ddc-6ace-462a-bc4a-ade18e4d273e",
        "service_label": "label-34"
      }
    }
  ]
}
```

#### How to Use
Valid States:
- CREATED
- DELETED
- UPDATED

#### Managed Service Instances vs User Provided Service Instances

Buh

### Querying For Events
- Using the query params to get only the events the billing system cares about as time moves forward.
- `after_guid`

#### Order of events
Events are sorted by internal database IDs. This order may differ from created_at. Events close to the current time should not be processed because other events may still have open transactions that will change their order in the results.

## External Warehouse
how an external system is meant to be used for warehousing the data
how old events will be deleted in ccdb which requires external warehousing

## Purge Endpoints
The purge endpoints are used to create initial events for all apps or service instances. The purge endpoints will delete ALL existing events, so it is important to use them **only once** when you are starting to record usage events for the first time.

### Apps
Destroys all existing events. Populates new usage events, one for each started app. All populated events will have a created_at value of current time. There is the potential race condition if apps are currently being started, stopped, or scaled. The seeded usage events will have the same guid as the app.

#### Endpoint
`POST /v2/app_usage_events/destructively_purge_all_and_reseed_started_apps`

### Services
Destroys all existing events. Populates new usage events, one for each existing service instance. All populated events will have a created_at value of current time. There is the potential race condition if service instances are currently being created or deleted. The seeded usage events will have the same guid as the service instance.

#### Endpoint
`POST /v2/service_usage_events/destructively_purge_all_and_reseed_existing_instances`
