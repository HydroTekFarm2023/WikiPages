```
The format will be as follows:
Topic: <topic title>
JSON format: {JSON Message: datatype}
----------------------------
Power Outlets
Out from frontend:
Topic: test_outlet_request/<device.id>
JSON format:
{
  "choice":number 
  "switch_status":number
}
Out from embedded system:
Topic: test_outlet_response/<device.id>
JSON format: 
{
  "choice":number
  "switch_status":number
}

Motors
Out from frontend:
Topic: test_motor_request/<device.id>
JSON format:
{
  "choice":number 
  "switch_status":number
}
Out from embedded system:
Topic: test_motor_response/<device.id>
JSON format: 
{
  "choice":number
  "switch_status":number
}

Float Switches
Out from frontend:
Topic: test_fs_request/<device.id>
JSON format:
{
  "choice":number 
  "switch_status":number
}
Out from embedded system:
Topic: test_fs_response/<device.id>
JSON format: 
{
  "choice":number
  "switch_status":number
}

Sensors
Out from frontend:
Topic: test_sensor_request/<device.id>
JSON format:
{
  "choice":string 
  "switch_status":number
}
Out from embedded system:
Topic: test_sensor_response/<device.id>
JSON format: 
{
  "choice":string
  "switch_status":number
}
```
