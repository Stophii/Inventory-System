# Inventory System

## Description

A video guide on [`inventory systems`](https://www.youtube.com/watch?v=lfokR_8JLME&t=68s) is located here if you want to follow along!

## Starting off

Every new Turbo project begins with a `lib.rs`, a `GameState` struct, and an impl block that defines the `new()` and `update()` functions. 

To set up a simple inventory system, create a new file named `player.rs`.

You can create a new file directly from Visual Studio Code (or any editor you prefer).

<img width="502" height="554" alt="Screenshot 2025-09-27 at 12 10 15 PM" src="https://github.com/user-attachments/assets/88326cb7-f49d-466f-990d-7eed5c211740" />

Name the file `player.rs`. 

At the very top of `player.rs`, add the following line so it can reference items from lib.rs:

```rust
use crate::*;
```

Next, at the top of `lib.rs`, add these lines:

```rust
use player::*;
mod player;
```
`mod player;` tells Rust to include the `player.rs` module, and `use player::*;` brings its public items into scope. The word “player” is used because of the filename `player.rs`.

> [!TIP]
> When you add more files, repeat the same pattern in `lib.rs`:
>
> ```rust
> use player::*;
> mod player;
> use {file_name}::*; // <-- Add this for new files!
> mod {file_name}; // <-- Add this for new files!
> ```

Now that we have a file that is recognized and enabled let's add structs for `player` and `item` into the file.

## Player and Item Structs

Inside the newly created `player.rs` file let's add:

```rust
#[turbo::serialize]
pub struct Player {
    pub hp: f32,
    pub mp: f32,
    pub money: f32,
    pub inventory: Vec<Item>,
}

#[turbo::serialize]
pub struct Item {
    pub name: String,
    pub description: String,
    pub quantity: u32,
}
```

These two structs contain some basic fields that you might have inside of a `player` and `item` struct.

In order to make an inventory system the `inventory` field is required as well as the `item` struct.

Now we need to add a `player::new()` function as well as a `player::use_item()` function, we'll do this inside a player `impl` block:

```rust
impl Player {
    pub fn new() -> Self {
        Self {
            hp: 100.0,
            mp: 100.0,
            money: 25.0,
            inventory: vec![
                Item {
                    name: "Hp Potion".to_string(),
                    description: "Increase your hp".to_string(),
                    quantity: 1,
                },
                Item {
                    name: "Mp Potion".to_string(),
                    description: "Increase your mp".to_string(),
                    quantity: 1,
                },
                Item {
                    name: "Limited Edition Turbi".to_string(),
                    description: "Do not sell this!".to_string(),
                    quantity: 1,
                }
            ]
        }
    }

    pub fn use_item(&mut self, index: usize) {
        let item = &mut self.inventory[index];
        if item.quantity > 0 {
            match item.name.as_str() {
                "Hp Potion" => {
                    self.hp += 20.0;
                    log!("Used HP Potion!");
                }
                "Mp Potion" => {
                    self.mp += 50.0;
                    log!("Used MP Potion!");
                }
                "Limited Edition Turbi" => {
                    self.money += 100.0;
                    log!("Sold the Turbi! How could you!");
                }
                _ => {}
            }
            item.quantity -= 1;
        } 
    }
}
```
> Make sure you make the function is public or `pub` otherwise you won't be able to initialize it in the next step!

Now that we have our `impl` and `structs` inside the `player.rs` we can move over to the `lib.rs` to initialize some fields.

> [!TIP]
> At this point I recommend running your project with `turbo run -w` from inside Visual Studio Code terminal
> or `turbo run -w nameofproject` from your system terminal.
> so you can view real-time updates with Cmd+S/Ctrl+S (saving) and Cmd+R/Ctrl+R (reloading), this is one of Turbos biggest strengths!

## Initializing GameState Fields & UI

In `GameState` we are going to add a `player` field and make it a `Player`. additionally we'll add a `selected_item` field and make that a `usize`.

```rust
#[turbo::game]
struct GameState {
    player: Player,
    selected_item: usize,
}
impl GameState {
    pub fn new() -> Self {
        // initialize your game state
        Self {
            player: Player::new(),
            selected_item: 0,
        }
    }
    pub fn update(&mut self) {
        // This is where your main game loop code goes
        // The stuff in this block will run ~60x per sec
        text!("Hello, world!!!");
    }
}
```

Intialize `player` to `Player::new()` and `selected_item` to `0`.

Now we need a basic UI to display the Player and all their stats (items, inventory, hp, etc.) Feel free to use mine or personalize your own.

> If you make your own just make sure to display the player's stats as well as a selector to indicate what item the `state.selected_item` is currently selecting!
```rust
fn ui(state: &mut GameState) {
    let hp = format!("hp: {}/{}", state.player.hp, state.player.hp);
    let mp = format!("mp: {}/{}", state.player.mp, state.player.mp);
    let money = format!("money: {}z", state.player.money);
    rect!(w = 240, h = 100, x = 10, y = 40, color = 0x000000ff, border_color = 0xffffffff, border_size = 1, border_radius = 2);

    text!(&hp);
    text!(&mp, y = 10);
    text!(&money, y = 20);
    text!("Inventory", x = 100, y = 45);

    for (i, item) in state.player.inventory.iter().enumerate() {
        let y = 70 + (i as i32) * 10;
        let line = format!("{} x{} - {}", item.name, item.quantity, item.description);
        text_box!(&line, x = 25, y = y);
    }

    let y = 71 + (state.selected_item as i32) * 10;
    rect!(x = 15, y = y, w = 6, h = 6, rotation = 45, color = 0xff0000ff);
}
```

If you don't know where to put it you can put the `ui` function just below your `GameState::update()` in the `lib.rs`.

You also need to add it to the update loop in order view the changes, go ahead and add some controls in as well.

```rust
    pub fn update(&mut self) {
        ui(self); // <- adding UI so it is visible
        let len = self.player.inventory.len();
        if len > 0 {
            let gp = gamepad::get(0);
            if gp.down.just_pressed() {
                self.selected_item = (self.selected_item + 1) % len;
            }
            if gp.up.just_pressed() {
                self.selected_item = (self.selected_item + len - 1) % len;
            }
        }
        if gamepad::get(0).a.just_pressed() {
            self.player.use_item(self.selected_item);
        }
    }
```
> [!TIP]
> Using `len()` will allow you to make expandable controls if your list gets bigger or smaller, I highly recommend it!

if you have your project running once you hit save (Cmd + S/Ctrl + S) you should see something like this

<img src="https://github.com/user-attachments/assets/645acdc2-10e5-4a9c-be65-cc5b2489693f" alt="Turbo" width="502">

Because of the controls and display you should be able to interact with your inventory and see the `qty` reduce with each item use.

## Ending Notes

With a `Z` press you can sell or use the items I've added to the inventory. If you want to add more items to the inventory just add them in the `player::new()` and make sure to give them a use in `player::use_item()`.

This has been a very basic inventory system using a selector and a few structs.

You can easily expand this system by adding an item database, shop interface, buy/sell/add item functions, and more!

Make sure to like and subscribe if you watched the video and join our [Discord](https://discord.gg/V5YWWvQvKW) for active help! 


