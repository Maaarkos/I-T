<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 13px; border-radius: 8px; border: 1px solid #444; line-height: 1.2; overflow-x: auto;">
           .o000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000o.
        .0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000.
      .00000000                                                                                     00000000.
     .00000                                 [ INTERFACE LAYER ]                                         00000.
    .0000                                   interface Loopback1                                           0000.
   .0000                                   (ip address 10.1.1.1)                                           0000.
   0000                                             |                                                       0000
  0000                                              | (ip unnumbered)                                        0000
  0000                                              v                                                        0000
 0000                                interface Virtual-Template 1                                             0000
 0000                                               |                                                         0000
 0000                                               | (tunnel protection)                                     0000
 0000                                               v                                                         0000
 0000                                 [ IPSEC / PHASE 2 LAYER ]                                               0000
 0000                            crypto ipsec profile FLEXVPN_PROFILE                                         0000
  0000                                              |                                                        0000
  0000                           +------------------+------------------+                                     0000
   0000                          |                                     |                                    0000
   .0000                (set transform-set)                    (set ikev2-profile)                         0000.
    .0000                        |                                     |                                  0000.
     .0000                       v                                     v                                 0000.
      .00000      crypto ipsec transform-set               [ IKEv2 / PHASE 1 LAYER ]                    00000.
        .00000         FLEXVPN_TRANSFORM              crypto ikev2 profile FLEXVPN_PROFILE            00000.
           .00000                                                      |                           00000.
              .00000                         +-------------------------+------------------+     00000.
                 .00000                      |                         |                  |  00000.
                    .00000            (keyring local)         (virtual-template) (aaa authorization)
                       .00000                |                         |                  |
                           .00000            v                         v                  v
                              .00000 crypto ikev2 keyring    Virtual-Template 1  crypto ikev2 auth
                                     FLEXVPN_KEYRING                               policy HUBPolicy
                                                                                          |
                                                                 +------------------------+------------------+
                                                                 |                        |                  |
                                                              (pool)                (route set)         (route set)
                                                                 |                        |                  |
                                                                 v                        v                  v
                                                       ip local pool FlexPool        10.1.1.1/32       ip access-list
                                                                                                        FlexTraffic
                                                                 \                        /
                                                                  \                      /
                                                                   \                    /
                                                                    |                  |
                                                                    |                  |
                                                                    |                  |
                                                          [ GLOBAL CRYPTO LAYER - The Roots ]
                                                           crypto ikev2 policy FLEXVPN_POLICY
                                                                           |
                                                                       (proposal)
                                                                           |
                                                                           v
                                                         crypto ikev2 proposal FLEXVPN_PROPOSAL
                                                                           |
                                                                        ___|___
                                                                       /       \
</pre>
</pre>