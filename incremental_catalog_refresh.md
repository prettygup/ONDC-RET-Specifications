### <ins>Incremental catalog refresh</ins>
* Incremental refresh is necessary for buyer NPs that cache catalogs and reflect changes since the last full catalog refresh;
* Every buyer NP that wants to consume incremental catalog changes should send a /search request through the gateway. This will be as an enhancement to the current /search API (for city) in the API contract;
* Buyer NP can send following config params in the /search request:
    * Full refresh:
        * payload_type - inline (response contains catalog), link (response points to link for accessing catalog);
    * Incremental refresh:
        * mode - "start", "stop";
        * start_time - start time for incremental refresh;
        * end_time - end time for incremental refresh;
* Following scenarios are possible:
    * Buyer NP requests 1-time incremental refresh:
        * start_time < Context.timestamp, end_time < Context.timestamp, end_time > start_time (means buyer NP pulls incremental updates from start_time to end_time in a single solicited response);
    * Buyer NP requests regular incremental refresh until stopped:
        * mode="start", start_time <= Context.timestamp (means seller NP pushes incremental updates from start_time at a frequency within a range of 1-30 mins). If no start_time provided, it defaults to Context.timestamp;
        * Seller NP should aggregate and push updates, at the same time, to all buyer NPs that have requested for incremental refresh;
        * If there are no incremental catalog updates, seller NP should not send empty catalog;
    * Buyer NP requests stopping of incremental refresh:
        * mode="stop";
* Buyer NPs can use the /search API to request for solicited or unsolicited incremental catalog refresh:
    * for solicited response, the response will have the same transaction_id & message_id as the request;
    * for unsolicited response, the response will have the same transaction_id but different message_id;
* To handle race condition where the buyer NP receives full & incremental refresh at the same time, versioning will be enabled for both types of refresh:
    * Full refresh will have a timestamp at the provider & location level and indicates when the snapshot was generated;
    * Incremental refresh will have a timestamp at provider / location / item level and indicates the time of the event;
* In case of race condition where the buyer NP receives full & incremental refresh at the same time, either of these approaches is feasible:
    * Buyer NP can process the full refresh and then initiate incremental refresh e.g. if full refresh at 8AM was ingested between 8-9AM, buyer NP can stop the incremental refresh before initiating the full refresh or choose to ignore all incremental refresh within this duration of catalog ingestion. After processing the full refresh, buyer NP can initiate incremental refresh based on the timestamp in full refresh;
    * Buyer NPs can process full & incremental refresh in sequence and use the timestamp to decide which incremental refresh updates to persist;
* To handle error conditions, i.e. buyer NP not responding or responding with a NACK, the seller NP should push catalog updates in multiple incremental refresh until ACK is received from all buyer NPs. Response format for NACK is [here](https://docs.google.com/document/d/1brvcltG_DagZ3kGr1ZZQk4hG4tze3zvcxmGV4NMTzr8/edit#heading=h.yypcl939fz3t)