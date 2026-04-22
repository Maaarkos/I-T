<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 13px; border-radius: 8px; border: 1px solid #444; line-height: 1.2; overflow-x: auto;">
<span style="color: #228B22;">                                       .o000000000000000000000000000000000000000000000000000o.
                                      .0000000000000000000000000000000000000000000000000000000000000.
                                   .0000000000000000000000000000000000000000000000000000000000000000000.
                                 .0000000000                                                 00000000000.
                                .0000000</span>                                                         <span style="color: #228B22;">0000000.
                               .000000</span>                 [ INTERFACE LAYER ]                         <span style="color: #228B22;">000000.
                               000000</span>                  interface Loopback1                          <span style="color: #228B22;">000000
                              000000</span>                  (ip address 10.1.1.1)                          <span style="color: #228B22;">000000
                              000000</span>                            <span style="color: #d2691e;">|</span>                                    <span style="color: #228B22;">000000
                              000000</span>                            <span style="color: #d2691e;">|</span> (ip unnumbered)                    <span style="color: #228B22;">000000
                              000000</span>                            <span style="color: #d2691e;">v</span>                                    <span style="color: #228B22;">000000
                               000000</span>             interface Virtual-Template 1                      <span style="color: #228B22;">000000
                               000000</span>                           <span style="color: #d2691e;">|</span>                                   <span style="color: #228B22;">000000
                                000000</span>                          <span style="color: #d2691e;">|</span> (tunnel protection)              <span style="color: #228B22;">000000
                                 000000</span>                         <span style="color: #d2691e;">v</span>                                 <span style="color: #228B22;">000000
                                  000000</span>            [ IPSEC / PHASE 2 LAYER ]                    <span style="color: #228B22;">000000
                                   000000</span>      crypto ipsec profile FLEXVPN_PROFILE             <span style="color: #228B22;">000000
                                    000000</span>                      <span style="color: #d2691e;">|</span>                              <span style="color: #228B22;">000000
                                     000000</span>            <span style="color: #d2691e;">+--------+--------+</span>                    <span style="color: #228B22;">000000
                                      00000</span>            <span style="color: #d2691e;">|</span>                 <span style="color: #d2691e;">|</span>                   <span style="color: #228B22;">00000
                                       0000</span>   (set transform-set) (set ikev2-profile)       <span style="color: #228B22;">0000
                                        000</span>            <span style="color: #d2691e;">|</span>                 <span style="color: #d2691e;">|</span>                 <span style="color: #228B22;">000
                                         00</span>            <span style="color: #d2691e;">v</span>                 <span style="color: #d2691e;">v</span>                <span style="color: #228B22;">00
                                          0</span>  crypto ipsec     [ IKEv2 / PHASE 1 ]        <span style="color: #228B22;">0</span>
                                             transform-set  crypto ikev2 profile
                                           FLEXVPN_TRANSFORM  FLEXVPN_PROFILE
                                                                 <span style="color: #d2691e;">|
                                                    +------------+------------+
                                                    |            |            |</span>
                                             (keyring local) (v-temp) (aaa authorization)
                                                    <span style="color: #d2691e;">|            |            |
                                                    v            v            v</span>
                                          crypto ikev2 keyring  V-Temp 1  crypto ikev2 auth
                                            FLEXVPN_KEYRING                policy HUBPolicy
                                                                                  <span style="color: #d2691e;">|
                                                                      +-----------+-----------+
                                                                      |           |           |</span>
                                                                   (pool)    (route set) (route set)
                                                                      <span style="color: #d2691e;">|           |           |
                                                                      v           v           v</span>
                                                            ip local pool    10.1.1.1/32  ip access-list
                                                              FlexPool                     FlexTraffic
                                                                      <span style="color: #d2691e;">\           |           /
                                                                       \          |          /
                                                                        \         |         /
                                                                         |        |        |
                                                                         |        |        |
                                                                         |        |        |</span>
                                                               [ GLOBAL CRYPTO LAYER - The Roots ]
                                                                crypto ikev2 policy FLEXVPN_POLICY
                                                                                  <span style="color: #d2691e;">|</span>
                                                                              (proposal)
                                                                                  <span style="color: #d2691e;">|
                                                                                  v</span>
                                                                crypto ikev2 proposal FLEXVPN_PROPOSAL
                                                                                  <span style="color: #d2691e;">|
                                                                               ___|___
                                                                              /       \</span>
</pre>