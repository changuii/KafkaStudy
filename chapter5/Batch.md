# 5.2 프로듀서의 배치

카프카에서는 토픽의 처리량을 높이기 위한 방법으로 토픽을 파티션으로 나눠 처리하며, 카프카 클라이언트인 프로듀서에서는 처리량을 높이기 위해 배치 전송을 권장합니다.  
따라서 프로듀서에서는 카프카로 전송하기 전, 배치(batch) 전송을 위해 토픽의 파티션별로 레코드들을 잠시 보관하고 있습니다.  

프로듀서의 배치 전송 방식은 단건의 메시지를 전송하는 것이 아니라 한 번에 다량의 메시지를 묶어서 전송하는 방법입니다.  
이러한 배치 전송은 불필요한 I/O를 줄일 수 있어 매우 효율적이며, 더불어 카프카의 요청 수를 줄여주는 효과도 있습니다.  

예를들어 프로듀서가 1000개의 메시지를 카프카로 전송한다면 1000번의 요청과 응답이 발생하지만 프로듀서가 100개의 메시지를 배치로 처리한다면 카프카와 프로듀서에서는 단 10번의 요청과 응답만 발생하게 됩니다.  

하지만 장점이 많다고 해서 무조건 배치 처리만 해야 하는 것은 아니며, 카프카를 사용하는 목적에 따라 처리량을 높일지, 아니면 지연 없는 전송을 해야 할지 선택을 해야 합니다.  

사용자가 프로듀서의 높은 처리량을 목표로 배치 전송을 설정하는 경우 주의해야 할 사항이 있습니다.  
바로 버퍼 메모리 크기가 충분히 커야 한다는 점입니다.  
즉 buffer.memory 크기가 반드시 batch.size보다 커야합니다.  

예를들어 토픽A가 3개의 파티션을 갖고 있고 batch.size는 기본값인 16KB라고 가정하겠습니다.  
그럼 프로듀서의 buffer.memory의 최소 크기는 16KB x 3이 되어야 합니다.  

프로듀서는 전송에 실패하면 재시도를 수행하는데, 이러한 부분까지 고려한다면 버퍼 메모리는 48KB보다 더 큰값으로 설정해야 합니다.  

배치 전송과 더불어 압축 기능을 같이 사용한다면, 프로듀서는 메시지들을 더욱 효율적으로 카프카로 전송할 수 있습니다.  
클라이언트를 포함해 카프카에서는 gzip, snappy, lz4, zstd 등의 압축 포맷을 지원합니다.  


