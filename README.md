import React, { useState } from 'react';
import { Upload, message, Button, Modal, List } from 'antd';
import { InboxOutlined } from '@ant-design/icons';

const { Dragger } = Upload;

const DragAndDropUploader = () => {
  const [isModalVisible, setIsModalVisible] = useState(false);
  const [csvData, setCsvData] = useState([]); // Holds the single column data
  const [fileContent, setFileContent] = useState(null); // Stores raw file content

  const props = {
    name: 'file',
    multiple: false,
    accept: '.csv',
    beforeUpload: (file) => {
      const isCsv = file.type === 'text/csv';
      if (!isCsv) {
        message.error('You can only upload CSV files!');
      } else {
        const reader = new FileReader();
        reader.onload = (e) => {
          setFileContent(e.target.result); // Save raw file content
        };
        reader.readAsText(file);
      }
      return isCsv || Upload.LIST_IGNORE; // Prevent upload if invalid
    },
    onChange(info) {
      const { status } = info.file;
      if (status === 'done') {
        message.success(`${info.file.name} file uploaded successfully.`);
      } else if (status === 'error') {
        message.error(`${info.file.name} file upload failed.`);
      }
    },
    onDrop(e) {
      console.log('Dropped files', e.dataTransfer.files);
    },
  };

  const handlePreview = () => {
    if (!fileContent) {
      message.warning('Please upload a CSV file first!');
      return;
    }

    // Parse CSV data manually
    const lines = fileContent.split('\n').filter((line) => line.trim() !== '');
    setCsvData(lines); // Set parsed single-column data
    setIsModalVisible(true); // Open modal
  };

  const handleModalClose = () => {
    setIsModalVisible(false); // Close modal
  };

  return (
    <div style={{ padding: '20px' }}>
      <Dragger {...props} style={{ padding: '20px', borderRadius: '10px' }}>
        <p className="ant-upload-drag-icon">
          <InboxOutlined />
        </p>
        <p className="ant-upload-text">Click or drag a CSV file to this area to upload</p>
        <p className="ant-upload-hint">Only single CSV file uploads are supported.</p>
      </Dragger>

      <div style={{ marginTop: '20px', textAlign: 'right' }}>
        <Button type="primary" onClick={handlePreview}>
          Preview
        </Button>
      </div>

      <Modal
        title="CSV Data Preview"
        visible={isModalVisible}
        onCancel={handleModalClose}
        footer={[
          <Button key="close" onClick={handleModalClose}>
            Close
          </Button>,
        ]}
      >
        <List
          bordered
          dataSource={csvData}
          renderItem={(item) => <List.Item>{item}</List.Item>}
        />
      </Modal>
    </div>
  );
};

export default DragAndDropUploader;