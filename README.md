# holding_pattern

Holding Pattern: 
A dependency-less way to enable offline-first SaaS apps.

Will have to think about several other aspects of this, like how to combine the local data with the remotely-grabbed data. Also, will need to address general localStorage saving of data.


For now, though:
- onSave:
  - check to see that localStorage exists for this browser
  - check to see if there is room on localStorage
  - if both are true
    - save the record as a JSON.stringified object, appending it to an existing hash of objects to save on the server
    - save a few additional fields:
      - fields_to_check_for_duplicates = []
      - this_instance_has_been_checked_against_the_remote_db = false
      - a_similiar_record_exists_in_the_remote_db = nil
      - this_instance_should_overwrite_the_record_in_the_remote_db = nil
    - the `fields_to_check_for_duplicates` record will use data-attributes in the form, and will enter those in as fields to check against the database
      - check configurable variables below for pattern
    - run the "saveLocalInstancesToRemoteDB" function
  - if localStorage is not available or is too full
    - save the data normally

- saveLocalInstancesToRemoteDB pseudocode:
- check to see if there's an internet connection
- if not: add a class to the html tag: "currently-offline"
  - if so:
    - remove "currently-offline" class from <html>
    - see if there are any records waiting in the Holding Pattern queue
    - for any with `this_instance_should_overwrite_the_record_in_the_remote_db`, attempt to save them remotely, then delete the local version
    - for any that don't have a `this_instance_should_overwrite_the_record_in_the_remote_db` value,
      - check each record against the remote database
        - change `this_instance_has_been_checked_against_the_remote_db` to TRUE
      - if the record DOES NOT EXIST in the remote DB:
        - save it to the database
        - delete it from localStorage
      - if the record DOES EXIST in the remote DB:
        - change `a_similiar_record_exists_in_the_remote_db` to TRUE
        - display a notice to the user:
          - "A record you entered while offline seems to exist on the database. What would you like to do?
            |   On Server   |     Local     |
            |   - details   |   - details   |
            |   - details   |   - details   |
            |   - details   |   - details   |
            |                               |
            |    [ Keep both/all records ]  |
            |                               |
            |               |               |
            |   Keep just   |   Keep just   |
            |  this record  |  this record  |
            |                               |
          - you might not show all the details, but if you do, probably best to use the fields in `fields_to_check_for_duplicates`
          - TODO: Think about how to handle multiple duplicate records; maybe checkboxes?

        - once the user picks the direction they want to go in:
          - if they pick "just the server record"
            - delete the local record
          - if they pick "just the local record"
            - change `this_instance_should_overwrite_the_record_in_the_remote_db` to TRUE
            - attempt to PATCH the record at the server with the values stored locally
            - if successful, delete the local record

- configurable variables:
  - frequency (in seconds) to check for data to store remotely: 30
  - name of data attribute to check against remote database: `data-holding-pattern-dupe-check`
  - use `default` or `custom` styling of alert box for duplicates: default
    - TODO: when building with JS, IF user has 'default' as variable, insert CSS section into HTML, scope all default CSS through parent class `.holding-pattern-alert`
- CSS classes to know about for custom theming:
  .holding-pattern-alert
    .holding-pattern-record
      .holding-pattern-record__title
      .holding-pattern-record__details
        .holding-pattern-record__detail--key
        .holding-pattern-record__detail--value
      .holding-pattern-record-button
      .holding-pattern-record-button--keep-individual
      .holding-pattern-record-button--keep-both (?)
      .holding-pattern-record-button--keep-all  (?)

