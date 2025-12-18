
# Home Assistant Integration

This guide will walk you through the process of setting up the Home Assistant integration with Harbor Scale. Easily collect and visualize your smart home data in **Grafana** with **Harbor Scale**.

## Features
- Push Home Assistant sensor data to Harbor Scale
- Automate data collection using REST commands
- View live dashboards in Grafana

## How to Use 

**Prerequisites**

Before starting, ensure you have:
- **Home Assistant** installed and running
- A working knowledge of **YAML** for Home Assistant configurations
- Smart home **sensors** or **devices** set up in Home Assistant

---

   **Step 1: Create a Harbor Scale Account**
1. **Sign up** at [Harbor Scale](https://harborscale.com/)
2. **Verify** your email and log in
3. **Create a Harbor**:
   - Click **Create Harbor** on your dashboard
   - Choose a **name** and select **General** as the type
   - Select the **Free** plan (or upgrade for more data capacity)
   - Click **Create**

4. **Retrieve your credentials**:
   - After creation, go to **View Details**
   - Note down:
     - `API Endpoint`
     - `API Batch Endpoint`
     - `API Key`
     - `Grafana Endpoint`
     - `Grafana Username`
     - `Grafana Password`

---

  **Step 2: Configure Home Assistant**
Edit your `configuration.yaml` to include the following REST command:

**configuration.yaml**
```yaml
rest_command:
  push_all_sensors_data:
    url: "YOUR_API_BATCH_ENDPOINT"
    method: POST
    headers:
      X-API-Key: "YOUR_API_KEY"
      Content-Type: "application/json"
    payload: >
      [
        {% for sensor in states.sensor 
            | selectattr('state', 'is_number') 
            | list %}
        {
          "time": "{{ now().utcnow().isoformat() }}Z",
          "ship_id": "{{ sensor.entity_id.split('.')[1].replace('_', ' ').title() }}",
          "cargo_id": "{{ sensor.attributes.friendly_name | default(sensor.entity_id.split('.')[-1].replace('_', ' ').title()) }}",
          "value": {{ sensor.state }}
        }{% if not loop.last %},{% endif %}
        {% endfor %}
      ]
  ```
---
**Step 3: Save & Restart**
Save your Yaml file and restart your home assistant
