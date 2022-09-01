---
title: "MyIKEv2 API"
linkTitle: "MyIKEv2 API"
weight: 140
description: >
---

MyIKEv2 provides two set of gRPC based APIs, 3rd party gRPC client could be developed with protobuf file in this doc.

* MyIKEv2 API: this API is used to control MyIKEv2 test instance 
* MyIKEv2 Daemon API: this API is used to control MyIKEv2 daemon instance 

## MyIKEv2 Test API
- getting running summary of MyIKEv2
- getting running summary of ping tasks
- clear ping stats
- list created IKE_SA
- dump a specified IKE_SA
- list CHILD_SA of a specified IKE_SA
- dump a specified CHILD_SA
- subscribe to MyIKEv2 events,with specified event filter
- stop MyIKEv2
- subscribe to MyIKEv2 final test result
- initiate CHILD_SA rekey

The listening address and port of API server could be configured via following options in setup file:

- apilistenaddr
- apilistenport

note: currently, MyIKEv2 API server doesn't support gRPC encryption/authentication;


The protobuf file:
```
// MyIKEv2 API
syntax = "proto3";
option go_package = "myikev2/api";
package api;

import "google/protobuf/timestamp.proto";
import "google/protobuf/duration.proto";

message Empty {}

//************** summary
message SummaryResp {
  uint32 Role =22; //1 is client, 2 is is gateway
  google.protobuf.Timestamp StartTime = 1;
  google.protobuf.Timestamp TestEndTime = 2;
  google.protobuf.Timestamp ActualTestEndTime = 26;
  google.protobuf.Timestamp CreationStartTime = 3;
  google.protobuf.Timestamp CreationFinishTime = 4;
  google.protobuf.Duration CreateDuration = 5;
  string SetupFileName = 6;
  uint32 NumOfCreatedTunnel = 7;
  float SetupRate = 8;
  uint64 Ikesa_state_init = 9;
  uint64 Ikesa_state_created = 10;
  uint64 Ikesa_state_established = 11;
  uint64 Ikesa_state_updatingaddr = 12;
  uint64 Ikesa_state_rekeying = 13;
  uint64 Ikesa_state_rekeyed = 14;
  uint64 Ikesa_state_closed = 15;
  uint64 Ikesa_state_closing = 16;
  uint64 Ikesa_state_dpd = 24;
  uint64 Ikesa_state_child_rekeying = 25;
  uint64 Ikesa_total = 17;
  uint64 Live_count = 18;
  uint64 Has_Child = 19;
  uint64 Created_live_count = 20;
  uint64 Configured_count = 21;
  uint64 Flapping_count = 23;
  uint32 Result = 27;
  string LastErrMsg = 28;
}

//************** get list of IKESA own SPI
message ListIKESAQuery {
  uint32 Start = 1; // start from zero
  uint32 Len = 2;   // 0 means return all
}

message IKESASummary {
  bytes PeerAddr = 1;
  uint32 PeerPort = 2;
  fixed64 OwnSPI = 3;
  uint32 State = 4;
  google.protobuf.Timestamp EstabTime = 5;

}

message ListIKESAResp { repeated IKESASummary SummaryList = 1; }


//************** get a list of all CHILD_SA own SPI of a given IKE_SA
message ListCHILDSAQuery {
  fixed64 IKEOwnSPI = 1;
}

message ListCHILDSAResp { repeated fixed32 OwnSPIList = 1; }

//************** dump CHILD_SA
message CHILDSAQuery { fixed32 OwnSPI = 1; }

message CHILDSADump {
  uint32 State = 1;
  fixed32 OwnSPI = 2;
  fixed32 PeerSPI = 3;
  bytes OwnAddr = 4;
  bytes PeerAddr = 5;
  fixed64 ParentIKESA = 6;
  google.protobuf.Timestamp EstabTime = 7;
  uint32 EncAlg = 8;
  uint32 KeyLen = 9;
  uint32 IntAlg = 10;
  google.protobuf.Duration LifeTime = 11;
  bytes SKei = 12;
  bytes SKer = 13;
  bytes SKai = 14;
  bytes SKar = 15;
  bool ESN = 16;
  bool TunnelMode = 17;
  uint32 ReplayWindowSize = 18;
  message TS {
    uint32 Type = 1;
    uint32 Protocol = 2;
    bytes StartAddr = 3;
    bytes EndAddr = 4;
    uint32 StartPort = 5;
    uint32 EndPort = 6;
  }
  repeated TS OwnTS = 19;
  repeated TS PeerTS = 20;
}



//************** dump IKE_SA
message IKESAQuery { fixed64 OwnSPI = 1; }

message IKESADump {
  uint32 State = 1;
  fixed64 OwnSPI = 2;
  fixed64 PeerSPI = 3;
  google.protobuf.Timestamp EstabTime = 4;
  bytes OwnAddr = 5;
  bytes PeerAddr = 6;
  uint32 PeerPort = 36;
  uint32 EncAlg = 7;
  uint32 KeyLen = 8;
  uint32 IntAlg = 9;
  uint32 PrfAlg = 10;
  uint32 OwnAuth = 11;
  uint32 PeerAuth = 12;
  uint32 MyIdType = 13;
  int32 HashAlgDS = 14;
  bool RSAPSS = 15;
  string PSK = 16;
  bool InitiateDPD = 17;
  bool ForceDPD = 18;
  google.protobuf.Duration DPDInterval = 19;
  google.protobuf.Duration LifeTime = 20;
  google.protobuf.Duration MarginTime = 21;
  bool Jitter = 35;
  bool InstallFastpath = 22;
  bool KeepChildHist = 23;
  bool KeepIKEHist = 24;
  bool EnableNATT = 25;
  google.protobuf.Duration NATTKeepaliveInterval = 26;
  google.protobuf.Timestamp LastRcvPktTime = 28;
  google.protobuf.Timestamp LastSendDPDTime = 29;
  bytes SKei = 30;
  bytes SKer = 31;
  bytes SKai = 32;
  bytes SKar = 33;
  uint32 CloseCode = 34;
}
//************** log
message EventFilter {
  uint32 Level=1;
  string keyword=2;
}
message MyIKEv2Event {
  uint32 Level =1;
  string Msg=2; 
  google.protobuf.Timestamp EventTime=3;
}

//************** ping stats request
message PingResultQuery {
  uint32 Start = 1; // start from zero
  uint32 Len = 2;   // 0 means return all
}

//*************** ping task stats
message PingResult {
  string LocalAddr =1;
  string RemoteAddr =2;
  uint64 TotalSentPkt=3;
  uint64 TotalRecvPkt=4;
}

message ListPingResult { 
  repeated PingResult ResultList = 1; 
  uint64 TotalSent = 2;
  uint64 TotalRecv =3;
}

//*************** Gateway address pool summary
message PoolUsageSummary {
  bytes V4StartAddr=1;
  bytes V6StartAddr=2;
  uint64 V4Assigned=3;
  uint64 V6Assigned=4;
}


message StopReq { bool Gracefully = 1; }

//*************** rekey child
message RekeyChildReq {
  fixed32 OwnSPI=1;
}

service MyIKEv2APISvc {
  rpc Stop(StopReq) returns (Empty);
  rpc GetSummary(Empty) returns (SummaryResp);
  rpc GetIKESA(IKESAQuery) returns (IKESADump);
  rpc ListIKESA(ListIKESAQuery) returns (ListIKESAResp);
  rpc ListCHILDSA(ListCHILDSAQuery) returns (ListCHILDSAResp);
  rpc GetCHILDSA(CHILDSAQuery) returns (CHILDSADump);
  rpc SubscrEvent(EventFilter) returns (stream MyIKEv2Event);
  rpc UpdateEventFilter(EventFilter) returns(Empty);
  rpc GetPingSummary(PingResultQuery) returns(ListPingResult);
  rpc ClearPingStats(Empty) returns (Empty);
  rpc GetPoolUsageSummary(Empty) returns(PoolUsageSummary);
  rpc NotifyFinalResult(Empty) returns (stream SummaryResp);
  rpc RekeyChild(RekeyChildReq) returns (Empty);

}
```

## MyIKEv2 Daemon API

```
// MyIKEv2 daemon API
syntax = "proto3";

option go_package = "myikev2/daemonapi";

package daemonapi;

import "myikev2/api/api.proto";

message Empty {}

message DefineMyIKEv2TestReq {
  string Setup=1; 
  string LogDir=2;
}

//for non-myikev2 test, like sswan
message DefineOtherTestReq {
  string SetupCMDs=1;
  string UpCMDs=2;
  string DestroyCMDs=3;
  string DataIf=4;
  string DataIfAddr=5; //this is a prefix
  
}

message DefineGenericTestReq {
  uint32 Type = 1; //1 myikev2, 2 other
  string Name = 2;
  DefineMyIKEv2TestReq MyIKEv2Test = 3;
  DefineOtherTestReq OtherTest =4;
  bool Override = 5;
}


message StatusReq {
  string Name = 1;
}

message StatusResp {
  uint32 State = 1;
}

message ListTestStatusEntry {
  string Name = 1;
  uint32 Type =2;
  bytes APIAddr =3;
  uint32 APIPort =4;
  api.SummaryResp Status =5;
  api.ListPingResult PingResults = 6;
}

message ListTestStatusResp {
  repeated ListTestStatusEntry results = 1;
}

message DestroyReq {
  string Name = 1;
  bool Gracefully =2;
}

message ClearPingStatsReq {
  string Name =1;
}


service MyIKEv2DaemonAPISvc {
  rpc Define(DefineGenericTestReq) returns (Empty);
  rpc Status(StatusReq) returns (StatusResp);
  rpc List(Empty) returns (ListTestStatusResp);
  rpc Destroy(DestroyReq) returns (Empty);
  rpc ClearPingStats(ClearPingStatsReq) returns (Empty);
}
```