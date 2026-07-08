# wallpaper

Role to manage wallpapers of Potos Linux Clients.

Installs the images listed in `wallpaper_images` into `/usr/share/backgrounds`
(pruning any unmanaged files there), registers them in the GNOME wallpaper
picker via `/usr/share/gnome-background-properties/potos.xml`, and sets the
first image as the system-wide default background for light and dark mode
through a glib gschema override
(`/usr/share/glib-2.0/schemas/zz-potos-background.gschema.override`).

## Example Playbook

As this role is tested via Molecule one can use [that
playbook](../../extensions/molecule/wallpaper/converge.yml) as a starting
point:

```yaml
---
- name: Converge
  hosts: all
  become: true
  gather_facts: true
  tasks:
    - name: Run the wallpaper role
      ansible.builtin.include_role:
        name: projectpotos.roles.wallpaper
```

## Role Variables

The default variables are defined in [defaults/main.yml](./defaults/main.yml):

```yaml
# List of files to be found under `wallpaper` to be added as background,
# first element is set as default
wallpaper_images: []
```

### Example

```yaml
wallpaper_images:
  - name: "Potos"
    filename: "potos.png"
    options: "zoom"
    pcolor: "#161b21"
    scolor: "#161b21"
    shade_type: "solid"
```

| variable   | Possible values                                                           | Description                                                             |
|------------|---------------------------------------------------------------------------|-------------------------------------------------------------------------|
| name       |                                                                           | How the wallpaper is named to the user                                  |
| filename   |                                                                           | Image file under `wallpaper/` to install                                |
| options    | "none", "wallpaper", "centered", "scaled", "stretched", "zoom", "spanned" | Determines how the image is rendered                                    |
| pcolor     | any hex rgb value e.g. #161b21                                            | Left or top color when drawing gradients, or the solid color.           |
| scolor     | any hex rgb value e.g. #161b21                                            | Right or bottom color when drawing gradients, not used for solid color. |
| shade_type | "horizontal", "vertical", and "solid"                                     | How to shade the background color.                                      |

Another option is to use `ansible-doc` to read the argument specification:

```sh
ansible-doc --type role projectpotos.roles.wallpaper
```

## Requirements

Files listed under `wallpaper_images` need to exist under `wallpaper/` in the
playbook's file search path.
