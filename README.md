# Govee Scene Light — Control4 Driver

**Version 3.7.0** · DriverWorks (Lua) · IP control · Proxy: `light_v2`
Author: Dillon Martine

Control Govee Wi-Fi lights from Control4 as a native light. The driver talks to the
[Govee Platform API](https://developer.govee.com/) and exposes your Govee devices as a
dimmable, colour-capable Control4 light that can also apply Govee scenes (built-in,
DIY and snapshots).

Because it binds the standard `light_v2` proxy, it behaves like any other Control4
light: it lives in a room, shows up in Navigator, responds to keypads, and can be used
in schedules, programming and Advanced Lighting scenes.

---

## Features

- **On / Off / Toggle / Dim** (1–100%) from Navigator, keypads and programming
- **Colour wheel** (full colour) and **colour temperature** slider (2000–9000 K)
- **Govee scenes** — pick from a dropdown in Composer, from the light's **Extras** tab in
  Navigator, or fire them from programming
- **Multi-device targeting** — one Control4 light can drive every device of a model,
  a single device, or a hand-picked list of devices (including different models)
- **State polling** — Control4 follows changes made in the Govee app, a voice assistant
  or a wall switch
- **Cached scene catalogue** that survives driver reloads and Director restarts
- **Built-in rate limiting** — requests are queued roughly 700 ms apart to stay under
  Govee's API limits

---

## Requirements

| Item | Requirement |
|---|---|
| Control4 OS | 3.3.0 or newer (needed for dropdown pickers in programming commands) |
| Tools | Composer Pro |
| Network | Controller needs outbound HTTPS to `openapi.api.govee.com` |
| Govee | Devices added to a Govee account, plus a Govee API key |
| Device support | Govee models that support the Platform API. Scene and colour support varies by model. |

**Getting a Govee API key:** in the Govee Home app go to **Profile → Settings → Apply for
API Key**. The key is emailed to you.

---

## Installation

1. In Composer Pro, open **Driver → Add or Update Driver** and select
   `govee_scene_light.c4z`.
2. In **System Design**, search for **Govee Scene Light** and add it to a room.
3. Select the driver, open the **Properties** tab, and paste your key into **API Key**.

No connections need to be bound. The `LIGHT` connection (5001) is the proxy binding and
is set up automatically. The three button-link connections are optional (see
[Connections](#connections)).

---

## Setup

Do these steps in order. Results appear in the **Last Result** property and in the Lua
output window (**Composer Pro → Tools → Lua**, then the driver's tab).

1. **Test API Key** (Actions tab) confirms the key works.
2. **Refresh Scene Catalogue** finds your devices and their scenes, then caches them.
3. **List Devices** prints your SKUs, device IDs, names and scene counts to the Lua
   output.
4. **Choose what this light controls:**
   - **Target SKU**: pick a model (e.g. `H6159  (3 devices)`). The light controls every
     device of that model.
   - Optionally, use **Device** to narrow it to one device of that SKU.
   - Or fill in **Target Devices** with a comma-separated list of names or IDs to control
     specific devices, including devices of different models.
5. **Reload Scene List** fills the **Scene** dropdown with the scenes available to your
   targets.
6. **Pick a Scene.** It applies straight away.

> **If the Target SKU dropdown still says "(refresh catalogue to load SKUs)"** after a
> refresh, your Director can't update property lists at runtime. Type the SKU into
> **Target SKU Override** instead. **Last Result** will tell you when this has happened.

---

## Properties

### Connection and targeting

| Property | Description |
|---|---|
| **Driver Version** | Read-only. |
| **API Key** | Required. Your Govee API key. Stored as a password field. |
| **Target SKU** | Govee model to control. Filled in after a catalogue refresh. The light controls every non-group device of this SKU. |
| **Target SKU Override** | Free-text SKU. Only used when the Target SKU dropdown is still showing its placeholder. Normally left empty. |
| **Device** | Narrows Target SKU to one device. Leave on *(all devices of this SKU)* to control all of them. |
| **Target Devices** | Comma-separated device names or IDs, e.g. `Patio Strip, AB:CD:12:34`. When filled in, it **overrides Target SKU and Device** and can mix models. Matching ignores case. |

### Scenes

| Property | Default | Description |
|---|---|---|
| **Scene List** | Common to all targets | *Common*: only scenes that **every** targeted device has. *Combined*: any scene **any** targeted device has. |
| **Scene** | — | Scenes available to the current targets. **Changing it applies the scene immediately.** |
| **Power On First** | Yes | Turns the lights on before sending a scene. Govee ignores scenes sent to a light that is off, so leave this on unless you handle power yourself. |
| **Duplicate Scene Names** | Use Lowest ID | Some models have two different scenes with the same name. *Use Lowest ID* picks one. *Refuse* returns an error instead. |
| **Scene Family** | Any | Limit scenes to `lightScene`, `diyScene` or `snapshot`. |
| **Catalogue Max Age** | 7 days | How long the cached catalogue is used before it is refreshed automatically (1 day / 7 days / 30 days / Never expire). |

### Status and diagnostics

| Property | Default | Description |
|---|---|---|
| **Poll Interval** | 5 minutes | How often to read the real light state from Govee (Off, 1, 2, 5, 10, 15, 30 min). |
| **Light Status** | — | Read-only, e.g. `2 of 3 on, 80% (14:35)`. Also lists unreachable devices. |
| **Extras Section Style** | 1 | Layout format for the Navigator Extras tab. Try 2, then 3, if the Extras tab doesn't appear. |
| **Debug Mode** | Off | Off / Print / Print and Log. |
| **Catalogue Status** | — | Read-only. Device and scene counts, and when the catalogue was last refreshed. |
| **Last Result** | Idle | Read-only. Result of the last action or command. |

---

## Actions (Composer Pro → Actions tab)

| Action | Purpose |
|---|---|
| **Test API Key** | Makes one call to check the key. |
| **Refresh Scene Catalogue** | Finds devices and scenes and caches them. |
| **Reload Scene List** | Refills the Scene dropdown for the current targets. |
| **List Devices** | Prints SKUs, device IDs, names and scene counts. |
| **List Scenes For SKU** | Prints every scene and its ID for a SKU. |
| **Set Scene** | Applies a scene by SKU and scene name. |
| **Set Scene (typed)** | Same, using free text. Partial names are accepted if they match only one scene. |
| **Set Scene By ID** | Applies a scene by SKU and numeric scene ID. Use this when names are ambiguous. |
| **Clear Catalogue** | Deletes the cached catalogue. |
| **Poll Now** | Reads the light state from Govee immediately. |
| **Force Online** | Tells Control4 the device is online and reports the current level. Use it if the light shows as disconnected. |
| **Report Light On / Off** | Updates the on/off state in the Control4 UI without sending anything to Govee. Use these to check that the UI updates. |
| **Report Test Colour** | Shows red on the colour wheel without sending anything to Govee. |
| **Push Lighting Extras** | Resends the Extras tab layout to Navigator. |
| **Dump Extras XML** | Prints the Extras XML to the Lua output. |

---

## Programming commands

In **Programming**, select the driver, then choose **Device Specific Command**:

| Command | Parameters | Notes |
|---|---|---|
| **Set Govee Scene** | SKU, Scene Name (dropdowns) | The main command. Dropdowns are filled from the cached catalogue. |
| **Set Govee Scene (typed)** | SKU, Scene Name (text) | Use if the dropdowns don't work on your Composer build. |
| **Set Govee Scene By ID** | SKU, Scene ID | Targets an exact scene ID. |
| **Refresh Govee Catalogue** | — | Good to run on a nightly schedule. |

Because the driver is a standard light, you can also use the usual light commands in
programming (On, Off, Toggle, Set Level, Ramp) and include it in Advanced Lighting scenes.

**Tip:** run **Refresh Scene Catalogue** once after installing so buttons respond
instantly. If a command runs before any catalogue exists, the driver builds the
catalogue first and then applies the scene. That works, but it takes a few seconds.

---

## How the light behaves

| Control4 action | What gets sent to Govee |
|---|---|
| **On** | Turns power on, then reapplies the selected Scene (if one is set). |
| **Off** | Turns power off. |
| **Toggle** | Off if the light is on, On if it is off. |
| **Set Level / Ramp** | Turns power on and sets brightness (1–100). Level 0 turns the light off. Govee doesn't ramp, so the change is immediate. |
| **Colour wheel** | Converted to RGB and sent as `colorRgb`. |
| **Colour temperature** | Sent as `colorTemperatureK` (2000–9000 K). |
| **Extras → Scene** | Applies the chosen Govee scene. |

**Controlling several devices as one light:** commands go to every targeted device. When
the driver polls, the Control4 light shows **On** if any device is on, at the brightness of
the brightest device that is on. A device that doesn't respond is listed in **Light Status**
but never makes the light show Off by itself.

**Scene IDs differ between devices:** the same scene name can have a different ID on each
device. The driver looks up the right ID for each device when it sends a scene.

**Govee groups** (e.g. `SameModeGroup`, `DreamViewScenic`) appear in device lists as
`[group]` but are never controlled, because they don't expose scenes or brightness.

---

## Navigator Extras tab

The light adds an **Extras** tab in Navigator with a **Govee Scenes** section. It holds a
scene list and a **Refresh Scenes** button. Picking a scene applies it.

If the Extras tab doesn't appear, change **Extras Section Style** to `2` and run **Push
Lighting Extras**. If it still doesn't appear, try `3`.

---

## Connections

| ID | Name | Type | Purpose |
|---|---|---|---|
| 5001 | LIGHT | `LIGHT_V2` proxy | Main light binding (set up automatically). |
| 300 | On Button Link | `BUTTON_LINK` | Optional keypad input. |
| 301 | Toggle Button Link | `BUTTON_LINK` | Optional keypad input. |
| 302 | Off Button Link | `BUTTON_LINK` | Optional keypad input. |

---

## Govee API usage

Base URL: `https://openapi.api.govee.com/router/api/v1`

| Endpoint | Used for |
|---|---|
| `GET /user/devices` | Finding devices |
| `POST /device/scenes` | Built-in light scenes |
| `POST /device/diy-scenes` | DIY scenes |
| `POST /device/control` | Power, brightness, colour, scenes |
| `POST /device/state` | Polling state |

Requests are queued and sent roughly 700 ms apart to stay within Govee's limits (about
2 requests/second per device and 12/second per account; 30 state requests/minute per
device). Polling once every five minutes uses very little of that.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| Light shows **disconnected** | Run **Force Online**. Check the API key with **Test API Key**. |
| **Target SKU** dropdown never fills | Type the SKU into **Target SKU Override**. Run **List Devices** to see your SKUs. |
| **Scene** dropdown is empty | Set a target, then run **Reload Scene List**. Make sure **Catalogue Status** shows a catalogue. |
| A scene is missing when controlling several devices | **Scene List** = *Common* only shows scenes every target has. Switch to *All targets combined*. |
| A scene "succeeds" but nothing changes | Make sure **Power On First** = Yes. Some models may expect a different scene value format; set **Debug Mode** to *Print and Log* and check the Lua output. |
| Nothing happens and Last Result warns about an unmatched selector | A name or ID in **Target Devices** didn't match a device. By design, the driver then controls no devices rather than guessing. Fix the spelling. |
| Control4 UI doesn't follow the lights | Use **Report Light On/Off** and **Report Test Colour** to check whether the UI updates without involving Govee. If they work, check **Poll Interval** and **Light Status**. |
| New commands don't appear in Programming | Composer caches driver files. Update the driver, then remove and re-add the device or restart Director. |
| No Extras tab | Change **Extras Section Style** to 2 or 3, then run **Push Lighting Extras**. |

For any issue, set **Debug Mode** to **Print and Log** and watch the Lua output window. The
driver always logs proxy commands it doesn't recognise, and every colour command, even
with Debug Mode off.

---

## Notes and limitations

- Govee doesn't support ramping, so brightness and colour changes happen immediately.
- Scene and colour support depends on the Govee model and its firmware.
- The light's state comes from polling, so changes made outside Control4 can take up to
  one **Poll Interval** to show.
- The cached catalogue is stored in the driver's persistent data. **Clear Catalogue**
  deletes it.

## Package contents

| File | Description |
|---|---|
| `driver.xml` | Driver definition: proxy, capabilities, properties, actions, commands |
| `driver.lua` | Driver logic |
| `README.txt` | Developer notes and version history |
