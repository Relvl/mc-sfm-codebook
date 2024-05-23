# Super Factory Manager Codebook
The mod allows you to transfer items, fluids, gases, and even power over your inventories and machines.
It can transfer as many things as inventories, machines, tanks and batteries slots/buffers are possible.
The mod do a transfers in one tick, every few (20 is minimum) ticks.

Sorry my English, it is not my main language ;)

## Basics
1. You need a controller, a disk, some inventory cables, and a label gun.
2. Connect all your inventories and machines with cables. Controller don't count as a cable. Diagonals are not connections.
3. Use a label gun, add/select a label, close the GUI, and label your things. It's allowed to label few inventories/machines same, and put few labels on same block.

## Hello world
Simplest program, that just move a dirt from one chest to another:
```sql
-- this is main loop. it's contents will execute every 20 ticks
every 20 ticks do
    -- 'input' is first and required keyword. at you can imagine, it marks things to input to the different place
    -- as a source thing you can just use item ID as default thing type for this example.
    input dirt from chest_a
        -- 'output' is another required keyword. it tries to place marked things to the target inventory. 
        output to chest_b
            -- 'forget' is forgets all marked before things and inputs. it's not necessary, but you want it if you write more than example. 
            -- just get used to using it after each operation.
            forget
    -- and finish the main loop
end
```
Also, you can inline your code if you found it clearer for yourself. This does the same as previous code:
```sql
every 20 ticks do
    -- same as before, just inlined. also there is specified thing type, just to show that it's possible
    -- also you can specify the mod: item:minecraft:dirt, item:mekanism:advanced_alloy
    -- you can hit F3+H keys same time while you in the main game screen to access to the advanced tooltips, that contains these IDs
    input item::dirt from chest_a output to chest_b forget
end
```
You can mark more than one source locations:
```sql
every 20 ticks do
    -- take some (all as possible) dirt. pretty easy.
    input dirt from chest_a
    -- and some cobby
    input cobblestone from chest_b
        -- here you can put _everything_ wildcard. it's unnecessary, there is a _default_ part, 'item::', that will be used if you don't write it
        output * to chest_c
            -- forgetting last operation is excessive, but why not? it's a good habit
            forget
end
```
You can mark *everything* to input, and output just few things:
```sql
every 20 ticks do
    -- mark every _item_ (as a default thing type) that contains in the inventory. there is no any tps overhead for this.
    input from chest_a
        -- you can specify the thing that will be outputted. for now it is item:minecraft:cobblestone
        output cobblestone to chest_b
        -- in some cases you should wrap the item ID in quotas or specify it is item. like this "redstone", that also is SFM keyword
        -- otherwise you can specify the item with it's thing type: item:minecraft:redstone or even item::redstone
        output "redstone" to chest_c
            forget
end
```
You can do a few operations at a time:
```sql
every 20 ticks do
    -- okay, usual operation
    input dirt from chest_a
        output to chest_b
            forget
    -- and another usual operation! this will be processed in the same tick as previous as they both in the same main loop!
    -- and also note that unrelated inventories may be processed a -> b, c -> d
    -- there is no limits, you can do a LOT of operations in the same loop, just don't forget to a forget to not to be get fucked lmao ;)
    input * from chest_c
        output item::obsidian to chest_d
            forget
end
```

## Forget
Okay, but what is the damn `forget` keyword??? It's forgets all previously marked inputs. Let's forget to add it:
```sql
every 20 ticks do
    -- okay, as always, take some cobby
    input cobblestone from chest_a
        -- and place it to another chest
        output to chest_b
            -- look, there is no any forget!
    -- now take some dirt
    input dirt from chest_c
        -- and place it to another one chest
        -- that's all? no! you don't forget chest_a! so, put some cobblestone from chest_a to chest_d also! is this a trashcan???
        output to chest_d
            -- and no forget again!
    -- take all the items you have
    input from chest_a
        -- and put some redstone to the interface (ofc you can label an interface! it's inventory too)
        output item::redstone to labeled_interface
    -- and last take some obsidian...
    input obsidian from chest_a
        -- and put it in the last chest
        -- remember? you don't forget a cobby from chest_a, dirt from chest_c, and EVERY items in chest_a 
        ---- (yes, there is collision in the input marks, but SFM is stupid enough to ignore it)
        -- and yes, put everything from chest_a, dirt from chest_c (don't care a cobblestone, it's count as everything)
        -- and there is no limits, it puts every available item stack to every available free/partial slot
        output to chest_x
    -- just don't forget to forget, if you don't understand what you does!
    -- rejecting a 'forget' keyword can be useful. but more often than not, you'd better have it.
end
```

## Sides
You can specify the side of the machine, which SFM should interact with:
```sql
every 20 ticks do
    -- take some (up to stack) redstone from interface, that provide it
    input item::redstone from labeled_interface
        -- and put it to the LEFT side of meka infuser. side must me configured as 'EXTRA' input!
        output to meka_infuser_red left side
            forget
    -- also take some iron ingots, and put it to the RIGHT side, which is configured as 'INPUT'
    -- if the interface/inventory can provide more than one stack at the same tick, and you using a meka factory, 
    ---- this will put all available iron ingots to all available machine slots!
    input iron_ingot from labeled_interface
        output to meka_infuser_red right side
            forget
    -- and take everything from the TOP side, which is configured as 'OUTPUT'. you don't need to enable eject/auto output on these machines
    input * from meka_infuser_red top side
        -- and put it to the interface
        output to labeled_interface
            forget
end
```
Some machines that allows you con configure their sides, *must* be accessed with specified sides! Meka, IF, Thermal - they will not work without specified sides!

## Fluids and power
Yup, you can move a fluids, gases, and even FE power:
```sql
every 20 ticks do
    -- okay, let's take some energy!
    -- be noticed that it takes as much energy as poser source can provide at a tick! better choice is meka ultimate cube. but you should access it with sides.
    -- set ALL the sides as 'input/output', and SFM will take x6 as output rate, and you can attach an energy source to it also! i'd assume to use Flux Networks.
    -- there is good practice to label every your machine with dedicated label. Yes, you can put few different labels on the same block.
    -- so there is 'labeled_meka_separator' and 'power_consumer' labels on single meka infuser.
    input fe:: from labeled_meka_cube each side
        -- okay, new thing and rule. you must specify the thing type, if it's not an item...
        -- so, we take fe::, and we put fe::, or we take gas:: and we put gas::, because default thing type is item::
        output fe:: to each power_consumer each side
            forget
    -- take some oxy gas. don't forget to specify a side 
    -- also small tip: almost every Meka machine can be configured to 'input/output' to the top side, and to every thing type: top in/out items, top in/out gases, etc. 
    -- just don't forget to disable ejection is you will stack the machines!
    input gas::oxygen from labeled_meka_separator top side
        -- to the destination you must specify only the thing type, it's unnecessary to specify the id...
        output gas:: to labeled_meka_prc
        -- ...but you can!
        -- the thing will be put in this order. first it tries to put in first output operation, then the second, etc.
        -- so if fill a machine, and rest gas will be put to the tank. if is possible ;)
        output gas::oxygen to labeled_meka_gas_tank
            forget
    -- same as gases, you can transfer a fluid
    input fluid::water from sink
        -- it is bad idea to feed a reactor through SFM, but it is first think lmao ;) DON'T DO THAT BOI! IT'S JUST AN EXAMPLE!
        output to reactor_port north side
            forget
end
```

## Program name
You can specify the name of your program. The disk contains it also will be named same:
```sql
name "my super program"
every 20 ticks do
    -- does nothing actually...
    -- yes, you can make a program without an instruction.
    -- yes, it's legal.
end
```

## Limits
You can move only desired items (or fluids, who cared?) at a loop tick:
```sql
-- ofc you can do the operations every minute (a minute if server's TPS is 20 lmao)
every 1200 ticks do
    input fluid:: from sink
        -- it's a millibucket is measurement of fluids, right?
        -- why 144? well... hello, GT boys!
        output 144 fluid:: to tank
            forget
    input from interface
        -- every 1200 ticks it will try to move just 4 items (or less, if there is no space in the destination)
        output 4 steel_ingot to some_destination
            forget
end
```
And a most cool thing - you may keep desired item (and other thing types ofc) count at a target:
```sql
every 20 ticks do
    input from interface
        -- there is 'retain' keyword. it takes as many items as needed to keep 20 items at the target inventory/side/slot
        output retain 20 honey_treat to incubator
            forget
    
end
```

## Variety, exception, and wildcards
You can use a list of items or labels, and even use few same labeled inventories. Also, you can use exception and wildcards:
```sql
every 20 ticks do
    -- there is 'except' keyword. it let SFM to take an items, except specified ones
    -- chest_a contains cobblestone, dirt, redstone, obsidian, and some mekanism alloys (idunno why, it's just an example)
    -- this operation takes redstone, obsidian, and every meka alloy (cuz it's all contains '_alloy' in their IDs) and put it to the interface
    input * except dirt,cobblestone,mekanism:*_alloy from chest_a
        output to interface
            forget
    -- you can use list of items, delimiter is comma
    -- you can use 'each' keyword to let it take from every same labeled inventories
    -- this operation takes only cobblestone and dirt, and don't care another items there
    input cobblestone,dirt from each chest_a
        -- same as items, you can use list of labels
        output 1 cobblestone to chest_b,chest_c
        -- you can use 'each' keyword to let it put to every same labeled inventories
        output 128 dirt to each chest_d
            forget
    -- and rest alloys to the another interface, why not?
    input mekanism:*_alloy from chest_a
        output to interface2
            forget
end
```

## Conditions
Ofc you can use a conditions to chest source/destination has/hasn't an item:
```sql
TODO
```