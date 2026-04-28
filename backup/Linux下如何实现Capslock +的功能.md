在Capslock+的[issue](https://github.com/wo52616111/capslock-plus/issues/90)中，我发现有人和我一样，已经离不开Capslock+，有在Linux下使用相同键位的需求。他们提到了Input Remapper，可以在linux下重映射键位，并提供了配置文件。

但他们回答的时间已经过于久远，Input Remapper的API已经经过了多次改动。于是我写了脚本以改造旧json，可用。遂[在此](https://github.com/wo52616111/capslock-plus/issues/90#issuecomment-2626240847)留下comment。

但是上面那个issue的东西只适用于特定型号笔记本电脑的内置键盘。回到换了SteamOS,使用2.4G无线键盘，又是另一套配置，上面issue中留下的config文件的所有带小写字母的Key Code全部不做数。无奈，只能手动修改以适配 `KEY_XX` 形式的Key Code.由于不明原因，新config文件无法上传，故而在此先将json写下，留作备忘：

```json
[
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 32,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_DOWN",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 31,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_LEFT",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 33,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_RIGHT",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 17,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_BACKSPACE",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 25,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_HOME",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 39,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_END",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 36,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_LEFTSHIF+KEY_LEFT",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 23,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_LEFTSHIFT+KEY_UP",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 37,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_LEFTSHIFT+KEY_DOWN",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 38,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_LEFTSHIFT+KEY_RIGHT",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 51,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_LEFTALT+KEY_LEFTSHIFT+KEY_RIGHT",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 35,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_LEFTCTRL+KEY_LEFTSHIFT+KEY_LEFT",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 30,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_LEFTCTRL+KEY_LEFT",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 34,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_LEFTCTRL + KEY_RIGHT",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 18,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_UP",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 14,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "k(KEY_HOME).m(KEY_LEFTSHIFT, k(KEY_END)).k(KEY_BACKSPACE)",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "if_single(key(KEY_ESC), None, timeout=300)",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 22,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_LEFTSHIFT+KEY_HOME",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 24,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_LEFTSHIFT+KEY_END",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 52,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_LEFTCTRL+KEY_LEFTSHIFT+KEY_RIGHT",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 57,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_ENTER",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 12,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_LEFTCTRL+KEY_PAGEUP",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 13,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_LEFTCTRL+KEY_PAGEDOWN",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 46,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_LEFTCTRL+KEY_C",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 47,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_LEFTCTRL+KEY_V",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 45,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_LEFTCTRL+KEY_X",
        "mapping_type": "key_macro"
    },
    {
        "input_combination": [
            {
                "type": 1,
                "code": 58,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            },
            {
                "type": 1,
                "code": 44,
                "origin_hash": "b79c789cc921e35f8ca0f76bb50132bb"
            }
        ],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_LEFTCTRL+KEY_Z",
        "mapping_type": "key_macro"
    }
]

```
