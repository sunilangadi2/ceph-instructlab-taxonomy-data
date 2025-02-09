# Scrub internals and diagnostics

## Scrubbing Behavior Table

+------------------------------+-----+------+---------+-------------+
| > Flags                      | n   | nos  | nodee   | noscrub/n   |
|                              | one | crub | p_scrub | odeep_scrub |
+==============================+=====+======+=========+=============+
| Periodic tick                | > S | > X  | > S     | > X         |
+------------------------------+-----+------+---------+-------------+
| Periodic tick after          | > D | > D  | > S     | > X         |
| osd_deep_scrub_interval      |     |      |         |             |
+------------------------------+-----+------+---------+-------------+
| Initiated scrub              | > S | > S  | > S     | > S         |
+------------------------------+-----+------+---------+-------------+
| Initiated scrub after        | > D | > D  | > S     | > S         |
| osd_deep_scrub_interval      |     |      |         |             |
+------------------------------+-----+------+---------+-------------+
| Initiated deep scrub         | > D | > D  | > D     | > D         |
+------------------------------+-----+------+---------+-------------+

-   X = Do nothing
-   S = Do regular scrub
-   D = Do deep scrub

## State variables

-   Periodic tick state is
    `!must_scrub && !must_deep_scrub && !time_for_deep`
-   Periodic tick after
    `osd_deep_scrub_interval state is !must_scrub && !must_deep_scrub && time_for_deep`
-   Initiated scrub state is
    `must_scrub && !must_deep_scrub && !time_for_deep`
-   Initiated scrub after `osd_deep_scrub_interval` state is
    `must_scrub && !must_deep_scrub && time_for_deep`
-   Initiated deep scrub state is `must_scrub && must_deep_scrub`

## Scrub Reservations

An OSD daemon command dumps total local and remote reservations:

    ceph daemon osd.<id> dump_scrub_reservations
