##### This script contains a few sections pulled from my "currently-in-development" Indie Studio Dashboard tool I am building for Shyborg Games.


#-------- Functions to securely query the API and receive response to interact with back end database. See example of query call in next section

func query_api(action: String, params: Dictionary, callback: Callable, use_post: bool = false) -> void:
	var http := HTTPRequest.new()
	add_child(http)
	print("🌐 Action requested: " + action)
	var headers := []
	var method := HTTPClient.METHOD_GET
	var url := "https://example.com/example.php"
	var body := ""

	if use_post:
		headers = ["Content-Type: application/json"]
		method = HTTPClient.METHOD_POST
		body = JSON.stringify(params)
		url += "?action=" + action
	else:
		var full_params := params.duplicate()
		full_params["action"] = action
		var client := HTTPClient.new()
		var query_string := client.query_string_from_dict(full_params)
		url += "?" + query_string

	# Connect with context for callback
	http.connect(
		"request_completed",
		Callable(self, "_on_query_response").bind(callback, action, http)
	)

	# Submit the request properly
	http.request(url, headers, method, body)


var verboseQuery = false
func _on_query_response(
	_result: int,
	response_code: int,
	_headers: PackedStringArray,
	body: PackedByteArray,
	callback: Callable,
	action: String,
	http_node: HTTPRequest
) -> void:
	var raw := body.get_string_from_utf8()
	if verboseQuery == true:
		print("🛰️ Raw response from [" + action + "]:\n" + raw)
	var parsed = JSON.parse_string(raw)
	if response_code == 200:
		if typeof(parsed) == TYPE_DICTIONARY:
			for key in parsed.keys():
				if parsed[key] == null:
					parsed[key] = "N/A"
			callback.call(parsed)
		elif typeof(parsed) == TYPE_ARRAY:
			for i in range(parsed.size()):
				if parsed[i] == null:
					parsed[i] = "N/A"
			callback.call(parsed)
		else:
			push_error("❌ Unexpected response type from [" + action + "]: " + str(parsed))
			print("❌ Unexpected response type from [" + action + "]: " + str(parsed))
	else:
		push_error("🚫 HTTP Error from [" + action + "]: " + str(response_code))
		print("🚫 HTTP Error from [" + action + "]: " + str(response_code))
	http_node.queue_free()
#endregion

#-------- Functions to instantiate task cards (newly added in Front End UI or loading from back end) | See example API - PHP in next section

func create_task_and_instantiate_new_card(index: int, source_button: OptionButton, bugfix: bool, panel: VBoxContainer) -> void:
	var new_task_card: TaskCard = TASK_CARD_INSTANCE.instantiate()
	new_task_card.add_to_group("TaskCards")
	new_task_card.mouse_filter = MOUSE_FILTER_IGNORE
	panel.add_child(new_task_card)
	panel.move_child(new_task_card, 0)
	initialize_new_task_card(new_task_card)
	# Set initial visual state
	var selected_text := source_button.get_item_text(index)
	var current_project = "Choose Project" if current_project_task == "" else current_project_task
	var type_index := get_option_index_by_text(new_task_card.task_type_option_button, selected_text)
	new_task_card.task_type_option_button.select(type_index)
	new_task_card.change_task_card_color(type_index)
	var project_index := get_option_index_by_text(new_task_card.task_project_option_button, current_project)
	new_task_card.task_project_option_button.select(project_index)
	new_task_card.task_elapsed_time_label.text = ""
	new_task_card.task_assigned_date_label.text = ""
	new_task_card.update_developer_name(true)
	new_task_card.bugfix_icon_texture_rect.visible = bugfix
	new_task_card.task_data_text_edit.text = "Type task description here..."
	new_task_card.currently_working = false
	new_task_card._on_edit_task_text_button_pressed()
	# Build backend payload
	var payload := get_task_payload_from_card(new_task_card)
	print(payload)
	# Submit using POST
	query_api(
		"create_task",
		payload,
		func(response):
			_on_create_task_response(new_task_card, payload, response),
		true
	)

func create_task_card_from_data(task_data: Dictionary) -> void:
	var new_task_card : TaskCard = TASK_CARD_INSTANCE.instantiate()
	new_task_card.add_to_group("TaskCards")
	new_task_card.mouse_filter = MOUSE_FILTER_IGNORE

	var selected_type = task_data.get("type", "N/A")
	var project_id := int(task_data.get("project_id", -1))
	var project_title = task_data.get("project_title", "Unassigned")
	var raw_developer_name = task_data.get("developer_name", "Unassigned")
	var description = task_data.get("description", "")
	var bugfix = task_data.get("bugfix", false)
	var date_assigned = str(task_data.get("date_assigned", ""))
	var date_assigned_full = str(task_data.get("date_assigned_full", ""))
	var currently_working = task_data.get("currently_working", false)
	var date_created = task_data.get("date_created", "")
	# Find matching developer_id from full name
	var matching_id := -1
	for dev_id in developer_data.keys():
		if developer_data[dev_id] == raw_developer_name:
			matching_id = dev_id
			break

	# Choose container panel based on assignment status
	var panel := unassigned_task_card_v_box_container
	if matching_id != -1:
		panel = currently_working_v_box_container if currently_working else currently_assigned_task_card_v_box_container

	panel.add_child(new_task_card)
	panel.move_child(new_task_card, 0)
	new_task_card.task_id = int(task_data.get("task_id", -1))

	# Populate dropdowns
	initialize_new_task_card(new_task_card)

	var type_index := get_option_index_by_text(new_task_card.task_type_option_button, selected_type)
	new_task_card.task_type_option_button.select(type_index)
	new_task_card.change_task_card_color(type_index)

	var project_index := get_option_index_by_text(new_task_card.task_project_option_button, project_title)
	new_task_card.task_project_option_button.select(project_index)

	for i in new_task_card.task_developer_option_button.get_item_count():
		var meta_id = new_task_card.task_developer_option_button.get_item_metadata(i)
		if int(meta_id) == matching_id:
			new_task_card.task_developer_option_button.select(i)
			break

	new_task_card.bugfix_icon_texture_rect.visible = bugfix
	new_task_card.task_data_text_edit.text = description
	new_task_card.date_assigned = date_assigned
	new_task_card.date_assigned_full = date_assigned_full
	new_task_card.task_assigned_date_label.text = new_task_card.get_elapsed_time_assigned_from_utc(str(date_assigned_full))
	new_task_card.currently_working = currently_working
	new_task_card.task_elapsed_time_label.text = new_task_card.get_elapsed_time_string(date_created)
	

	new_task_card.original_task_data = {
		"type": selected_type,
		"project_title": project_title,
		"project_id": project_id,
		"developer_id": matching_id,
		"description": description,
		"bugfix": bugfix,
		"date_assigned": date_assigned,
		"date_assigned_full": date_assigned_full,
		"currently_working": currently_working,
		"date_created": date_created  # ← store the raw timestamp for calculations
}

func _on_create_task_response(new_task_card: TaskCard, payload: Dictionary, response: Variant) -> void:
	if typeof(response) == TYPE_DICTIONARY and response.has("task_id"):
		new_task_card.task_id = int(response["task_id"])
		cached_tasks[new_task_card.task_id] = normalize_ts(response.get("last_updated", ""))
		# Snapshot original task data to avoid sync collisions
		new_task_card.original_task_data = {
			"type": payload.get("type", "N/A"),
			"project_title": payload.get("project_title", "Unassigned"),
			"project_id": payload.get("project_id", -1),
			"developer_id": payload.get("developer_id", -1),
			"description": payload.get("description", ""),
			"bugfix": payload.get("bugfix", "false") == "true",
			"date_assigned": format_neonDB_date_to_short_date(payload.get("date_assigned", "")),
			"date_assigned_full": payload.get("date_assigned_full", ""),
			"date_created": response.get("date_created", ""),
			"currently_working": payload.get("currently_working", "false") == "true"
		}
		print("✅ Task created:", new_task_card.original_task_data)
	else:
		push_error("❌ Task creation failed: " + str(response))


func get_task_payload_from_card(card: Node) -> Dictionary:
	var selected_type : String = card.task_type_option_button.get_item_text(card.task_type_option_button.get_selected())
	var selected_project : String = card.task_project_option_button.get_item_text(card.task_project_option_button.get_selected())
	var selected_project_id := int(card.task_project_option_button.get_item_metadata(card.task_project_option_button.get_selected()))
	var selected_developer_id = int(card.task_developer_option_button.get_item_metadata(card.task_developer_option_button.get_selected()))
	if selected_developer_id == -1:
		selected_developer_id = ""
	var selected_description : String = card.task_data_text_edit.text
	var selected_bugfix := str(card.bugfix_icon_texture_rect.visible)
	var selected_date_assigned = card.date_assigned
	var selected_date_assigned_full = card.date_assigned_full
	if selected_date_assigned == "" or selected_date_assigned == null:
		selected_date_assigned = "null"
	if selected_date_assigned_full == "" or selected_date_assigned_full == null:
		selected_date_assigned_full = "null"
	var currently_working := str(card.currently_working)
	return {
		"type": selected_type,
		"project_id": selected_project_id,
		"project_title": selected_project,
		"developer_id": selected_developer_id,
		"description": selected_description,
		"bugfix": selected_bugfix,
		"currently_working": currently_working,
		"date_assigned": selected_date_assigned,
		"date_assigned_full": selected_date_assigned_full
	}



#-------- Example of API "Action" when querying back end for task data

switch ($action) {
/* #region get_tasks */
    case 'get_tasks':
        #// ✅ Extract optional URL flags securely
        #// These are query parameters like ?include_completed=true
        #// ?? 'false' provides a fallback if the param isn't present
        #// === 'true' ensures strict string matching
        
        #// Ensures only strict 'true' gets treated as true — avoids '' or garbage input
        $include_completed = get_boolean_param('include_completed');
        $include_discarded = get_boolean_param('include_discarded');

        #// ✅ Define SQL query using parameter placeholders: $1 and $2
        #// These placeholders are **not** interpolated directly — they're replaced safely via binding
        #// This eliminates SQL injection risk and avoids manual escaping
        $query = "
            SELECT 
                t.task_id,                      -- Unique identifier for the task
                t.project_id,                   -- Foreign key for the related project
                p.project_title,                -- Project name from the projects table
                COALESCE(d.dev_name, 'Unassigned') AS developer_name, -- Developer name or fallback
                COALESCE(t.date_assigned::text, '') AS date_assigned, -- Assigned date as text, fallback empty
                t.date_assigned_full,           -- UTC full timestamp
                t.type,                         -- Task type (feature, bug, etc.)
                t.description,                  -- Task summary
                t.bugfix,                       -- Boolean: is bugfix
                t.currently_working,            -- Boolean: is active
                t.last_updated,                 -- Timestamp of last update
                t.date_created,                 -- Original creation time
                t.completed,                    -- Boolean: is completed
                t.discarded                     -- Boolean: is discarded
            FROM tasks t
            JOIN projects p ON t.project_id = p.project_id               -- Inner join: projects must match
            LEFT JOIN developers d ON t.developer_id = d.developer_id   -- Left join: developers may be null
            WHERE 
                ($1::boolean = TRUE OR t.completed = FALSE)
                AND 
                ($2::boolean = TRUE OR t.discarded = FALSE)
        ";

        #// ✅ Run query with parameters bound separately
        #// pg_query_params ensures input is sanitized and cannot break SQL structure
        #// $1 and $2 correspond to these array values:
        $result = pg_query_params($conn, $query, [$include_completed, $include_discarded]);

        #// 🧠 Handle results: check for valid result set and rows
        if ($result && pg_num_rows($result) > 0) {
            $tasks = []; // Empty array to collect rows

            while ($row = pg_fetch_assoc($result)) {
                #// Convert PostgreSQL booleans ('t'/'f') into true/false
                $row['bugfix'] = ($row['bugfix'] === 't');
                $row['currently_working'] = ($row['currently_working'] === 't');

                #// Add processed row to task list
                $tasks[] = $row;
            }

            #// Output task array as JSON — frontend can parse this easily
            echo json_encode($tasks);
        } else {
            #// No tasks found — return empty array
            echo json_encode([]);
        }

        #// End of case block in switch structure
        break;
/* #endregion */
