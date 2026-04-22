<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 13px; border-radius: 8px; border: 1px solid #444; line-height: 1.2; overflow-x: auto;">
                                [ INTERFACE LAYER ]
                                interface Loopback1
                                (ip address 10.1.1.1)
                                         ^
                                         | (ip unnumbered)
                                         |
                          interface Virtual-Template 1
                                         |
                                         | (tunnel protection)
                                         v
                            [ IPSEC / PHASE 2 LAYER ]
                       crypto ipsec profile FLEXVPN_PROFILE
                                         |
                       +-----------------+-----------------+
                       |                                   |
              (set transform-set)                  (set ikev2-profile)
                       |                                   |
                       v                                   v
        crypto ipsec transform-set             [ IKEv2 / PHASE 1 LAYER ]
             FLEXVPN_TRANSFORM                crypto ikev2 profile FLEXVPN_PROFILE
                                                           |
                                     +---------------------+---------------------+
                                     |                     |                     |
                              (keyring local)     (virtual-template)    (aaa authorization)
                                     |                     |                     |
                                     v                     v                     v
                           crypto ikev2 keyring    Virtual-Template 1  crypto ikev2 authorization
                             FLEXVPN_KEYRING                                policy HUBPolicy
                                                                                 |
                                                           +---------------------+---------------------+
                                                           |                     |                     |
                                                      (pool)             (route set interface) (route set access-list)
                                                           |                     |                     |
                                                           v                     v                     v
                                                 ip local pool FlexPool      10.1.1.1/32       ip access-list standard
                                                                                                     FlexTraffic

=========================================================================================================================
                                [ GLOBAL CRYPTO LAYER - Unattached ]
                                 crypto ikev2 policy FLEXVPN_POLICY
                                                 |
                                             (proposal)
                                                 |
                                                 v
                                crypto ikev2 proposal FLEXVPN_PROPOSAL
</pre>