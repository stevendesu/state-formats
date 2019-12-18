# Method 5 ?

This is what I'm trying to figure out. Some perfect balance between methods 2
(which automatically prevented duplicates) and 5 (which was normalized and
therefore prevented a lot of weird edge cases)

In a perfect world this state would also make it easy to detect namespace
collisions, because I'm getting sick of how hard it is to detect those. I want
to throw up a red flag if you give a relation the same name as an existing
field, but that's **hard** for some reason. Does it have to be? I think not

## State Format

```
-- In progress
```
