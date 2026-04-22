<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 13px; border-radius: 8px; border: 1px solid #444; line-height: 1.2; overflow-x: auto; white-space: pre;">
<span style="color: #228B22;">                                          00000000000000000000000000000000000000000000000000000000000000000000000000000000
                                     00000                                                                                00000
                                 00000                                                                                        00000
                              00000                                                                                              00000
                           00000                                                                                                    00000
                        00000                                   </span><span style="color: #00ff00;">interface loopback 1</span><span style="color: #228B22;">                                                   00000
                      00000                               </span><span style="color: #00ff00;">ip address 10.1.1.1 255.255.255.255</span><span style="color: #228B22;">                                        00000
                    00000                                                 </span><span style="color: #d2691e;">|</span><span style="color: #228B22;">                                                            00000
                  00000                                                   </span><span style="color: #d2691e;">v</span><span style="color: #228B22;">                                                              00000
                00000                                   </span><span style="color: #00ff00;">interface virtual-template 1 type tunnel</span><span style="color: #228B22;">                                           00000
               00000                                            </span><span style="color: #00ff00;">ip unnumbered loopback1</span><span style="color: #228B22;">                                             00000
              00000                                 </span><span style="color: #00ff00;">tunnel protection ipsec profile FLEXVPN_PROFILE</span><span style="color: #228B22;">                                  00000
             00000                                                        </span><span style="color: #d2691e;">|</span><span style="color: #228B22;">                                                           00000
            00000                                                         </span><span style="color: #d2691e;">v</span><span style="color: #228B22;">                                                            00000
           00000                                        </span><span style="color: #00ff00;">crypto ipsec profile FLEXVPN_PROFILE</span><span style="color: #228B22;">                                            00000
          00000                                         </span><span style="color: #00ff00;">set transform-set FLEXVPN_TRANSFORM</span><span style="color: #228B22;">                                              00000
         00000                                          </span><span style="color: #00ff00;">set ikev2-profile FLEXVPN_PROFILE</span><span style="color: #228B22;">                                                 00000
        00000                                                </span><span style="color: #d2691e;">/                         \</span><span style="color: #228B22;">                                                   00000
       00000                                                </span><span style="color: #d2691e;">/                           \</span><span style="color: #228B22;">                                                   00000
      00000                                                </span><span style="color: #d2691e;">v                             v</span><span style="color: #228B22;">                                                   00000
     00000       </span><span style="color: #00ff00;">crypto ipsec transform-set FLEXVPN_TRANSFORM</span><span style="color: #228B22;">             </span><span style="color: #00ff00;">crypto ikev2 profile FLEXVPN_PROFILE</span><span style="color: #228B22;">                                00000
    00000        </span><span style="color: #00ff00;">esp-aes 256 esp-sha-hmac</span><span style="color: #228B22;">                                 </span><span style="color: #00ff00;">match identity remote address 0.0.0.0</span><span style="color: #228B22;">                                00000
   00000         </span><span style="color: #00ff00;">mode tunnel</span><span style="color: #228B22;">                                              </span><span style="color: #00ff00;">authentication remote pre-share</span><span style="color: #228B22;">                                       00000
  00000                                                                   </span><span style="color: #00ff00;">authentication local pre-share</span><span style="color: #228B22;">                                         00000
  00000                                                                   </span><span style="color: #00ff00;">keyring local FLEXVPN_KEYRING</span><span style="color: #228B22;">                                          00000
 00000                                                                    </span><span style="color: #00ff00;">aaa authorization group psk list FlexAuth HUBPolicy</span><span style="color: #228B22;">                     00000
 00000                                                                    </span><span style="color: #00ff00;">virtual-template 1</span><span style="color: #228B22;">                                                      00000
 00000                                                                      </span><span style="color: #d2691e;">/                       \</span><span style="color: #228B22;">                                             00000
 00000                                                                     </span><span style="color: #d2691e;">/                         \</span><span style="color: #228B22;">                                            00000
 00000                                                                    </span><span style="color: #d2691e;">v                           v</span><span style="color: #228B22;">                                           00000
 00000                                     </span><span style="color: #00ff00;">crypto ikev2 keyring FLEXVPN_KEYRING</span><span style="color: #228B22;">       </span><span style="color: #00ff00;">crypto ikev2 authorization policy HUBPolicy</span><span style="color: #228B22;">                 00000
 00000                                     </span><span style="color: #00ff00;">peer FLEVPNPeers</span><span style="color: #228B22;">                           </span><span style="color: #00ff00;">pool FlexPool</span><span style="color: #228B22;">                                               00000
 00000                                     </span><span style="color: #00ff00;">address 0.0.0.0 0.0.0.0</span><span style="color: #228B22;">                    </span><span style="color: #00ff00;">route set interface</span><span style="color: #228B22;">                                         00000
  00000                                    </span><span style="color: #00ff00;">pre-shared-key local cisco123</span><span style="color: #228B22;">              </span><span style="color: #00ff00;">route set access-list FlexTraffic</span><span style="color: #228B22;">                          00000
  00000                                    </span><span style="color: #00ff00;">pre-shared-key remote cisco123</span><span style="color: #228B22;">                   </span><span style="color: #d2691e;">/               \</span><span style="color: #228B22;">                                    00000
   00000                                                                                   </span><span style="color: #d2691e;">/                 \</span><span style="color: #228B22;">                                  00000
    00000                                                                                 </span><span style="color: #d2691e;">v                   v</span><span style="color: #228B22;">                                00000
     00000                                                                  </span><span style="color: #00ff00;">ip local pool FlexPool</span><span style="color: #228B22;">      </span><span style="color: #00ff00;">ip access-list standard FlexTraffic</span><span style="color: #228B22;">   00000
      00000                                                                 </span><span style="color: #00ff00;">10.1.1.2 10.1.1.254</span><span style="color: #228B22;">         </span><span style="color: #00ff00;">permit 10.10.1.0 0.0.0.255</span><span style="color: #228B22;">           00000
        0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000</span>
                                                                     <span style="color: #d2691e;">\        |        /
                                                                      \       |       /
                                                                       \      |      /
                                                                        |     |     |
                                                                        |     |     |
                                                                       /      |      \
                                                                      /       |       \
                                                                     |        |        |</span>
                                                                     <span style="color: #00ff00;">crypto ikev2 policy FLEXVPN_POLICY
                                                                     proposal FLEXVPN_PROPOSAL</span>
                                                                             <span style="color: #d2691e;">|
                                                                             v</span>
                                                                     <span style="color: #00ff00;">crypto ikev2 proposal FLEXVPN_PROPOSAL
                                                                     encryption aes-cbc-256
                                                                     integrity sha256
                                                                     group 14</span>
                                                                          <span style="color: #d2691e;">___|___
                                                                         /       \</span>
</pre>