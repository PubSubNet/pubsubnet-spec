## Theoretical and specific structures in PSN

1. U3ID

Universally Unique User Identifier: 128-bit (16 byte) pseudorandomly generated integer stored / displayed as base256, used to uniquely identify users.

If the least significant bit (LSB) of the most significant nibble (MSN) is 0 (i.e. the MSN is an even number), the number is a U3ID. Otherwise, if the MSN is an odd number, the number is a UCID (see next).

2. UCID

Universally Unique Content Identifier: like a U3ID but used to uniquely idenfity content and "things", like items, boards, feeds, &c.

If the LSB of the MSN is 1 (i.e. the MSN is an odd number), the number is a UCID. Otherwise, if the MSN is an even number, the number is a U3ID (see prev).

1. Board

Collection of Items.

```c
ucid board_id
u3id creator_id?
string board_name
bool taggable
strings board_tags
bool fully_hashed, fully_encrypted
kind_of privacy
u3ids owners, operators, users, observers
items contents

// "pinned" attributes
string username, pretty_name
image avatar, background, cover
text style

// security attributes
challenge modify_challenge
```

2. Item

A resource. It may have content (segments), non-display attachments, and sandboxed scripts and style for polls, etc.

Each segment is base64'd, compressed, base64'd again, glued with record separators, and re-base64'd

```c
manifest segments_manifest
string segments, attatchments
string hash, pubkey

u3id author_id
ucid item_id

// optional republish data...

// metadata
dt cr, ul, fs?, ft
dt aa, da
bool encrypted, signed
```

3. Event

4. User

5. Hub

7. Local instance

