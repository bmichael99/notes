Previously in [[JLab OMS (FOMS)]] all actions in [[FOMS Router|Routers]] were single php files for single action types. This lead to a lot of duplicated code since there were many actions just for moving orders to different destinations.

Our new system allows one action type to have "payloads" of configuration.

Before
```
Actions: move_to_fba_us, force_move_to_fba_us, move_to_logiwa, etc...
```

After
```
Action: move, destination: <logiwa, fba_us>, force: true
```

This makes there be far fewer files to maintain this code, and much easier to create and maintain the [[FOMS Rule|Rules]] from the frontend.